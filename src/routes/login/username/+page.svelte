<script lang="ts">
    import AuthCheck from "$lib/components/AuthCheck.svelte";
    import { db , user, userData } from "$lib/firebase";
    import { doc , getDoc , writeBatch } from "firebase/firestore";
    
    let username = "";
    let loading = false;
    let isAvaible =false;

    const re = /^(?=[a-zA-Z0-9._]{3,16}$)(?!.*[_.]{2})[^_.].*[^_.]$/;
  
    $: isValid = username?.length > 2 && username.length < 16 && re.test(username);
    $: isTouched = username.length > 0;
    $: isTaken = isValid && !isAvaible && !loading


    let debounceTimer:NodeJS.Timeout;
    // Αυτή η συνάρτηση ελέγχει τη διαθεσιμότητα ενός όνομας χρήστη στη βάση δεδομένων.
    async function checkAvaibility(){
        isAvaible = false;
        clearTimeout(debounceTimer);
         // Καθαρίζει τον χρονόμετρο debounce, προστατεύοντας από πολλαπλές εκτελέσεις της συνάρτησης σε συνέχεια.
        loading = true;	

         // Ορίζει την μεταβλητή loading σε true, υποδεικνύοντας ότι η επιβεβαίωση του όνοματος χρήστη είναι σε εξέλιξη.
        debounceTimer = setTimeout(async ()=>{
             // Εκτυπώνει ένα μήνυμα στην κονσόλα για να ενημερώσει τον προγραμματιστή ότι ελέγχεται το όνομα χρήστη.
            console.log("Checking username", username);
            // Δημιουργεί μια αναφορά στο έγγραφο με το όνομα χρήστη στη βάση δεδομένων.
            const ref = doc(db, "usernames", username);
            // Ελέγχει αν το έγγραφο υπάρχει στη βάση δεδομένων.
            const exists = await getDoc(ref).then((doc) => doc.exists());
            
            isAvaible = !exists;
            loading = false;
        },500);
        
    }

    // Αυτή η συνάρτηση επιβεβαιώνει το όνομα χρήστη και δημιουργεί ή ενημερώνει δύο εγγραφές στη βάση δεδομένων.
    async function confirmUsername() {
        console.log("confirming username", username);
         // Δημιουργεί ένα batch για να εκτελέσει πολλαπλές εγγραφές σε μια μόνη λειτουργία
        const batch = writeBatch(db);
        // Ορίζει το όνομα χρήστη στο έγγραφο με το όνομα χρήστη στη βάση δεδομένων, συνδέοντας το με το uid του χρήστη.
        batch.set(doc(db,"usernames",username),{ uid: $user?.uid});
          // Δημιουργεί ή ενημερώνει το έγγραφο του χρήστη με τις πληροφορίες του, συμπεριλαμβανομένων του ονόματος χρήστη, της φωτογραφίας, της δημοσιευμένης κατάστασης, της βιογραφίας και των συνδέσμων.
        batch.set(doc(db,"users",$user!.uid),{
            username,
            photoURL: $user?.photoURL ?? null,
            published:true,
            bio:'',
            links:[
                {
                    title:'',
                    url:'',
                    icon:''
                }
            ]
        });

        await batch.commit();
        username='';
        isAvaible=false;
    }

</script>

<AuthCheck>
    {#if $userData?.username}
    <p class="text-lg">
      Your username is <span class="text-success font-bold"
        >@{$userData.username}</span
      >
    </p>
    <p class="text-sm">(Usernames cannot be changed)</p>
    <a class="btn btn-primary" href="/login/photo">Upload Profile Image</a>
   {:else}
    <form class=" w-2/5" on:submit|preventDefault ={confirmUsername}>
        <input 
            type="text"
            placeholder="username"
            class="input w-full"
            bind:value={username}
            on:input={checkAvaibility}
            class:input-error={(!isValid && isTouched)}
            class:input-warning={isTaken}
            class:input-success={isAvaible && isValid && !loading}
        />
        <div class="my-4 min-h-16 px-8 w-full">
            {#if loading}
              <p class="text-secondary">Checking availability of @{username}...</p>
            {/if}
        
            {#if !isValid && isTouched}
              <p class="text-error text-sm">
                must be 3-16 characters long, alphanumeric only
              </p>
            {/if}
        
            {#if isValid && !isAvaible && !loading}
              <p class="text-warning text-sm">
                @{username} is not available
              </p>
            {/if}
        
            {#if isValid && !isTaken && !loading}
              <button class="btn btn-success">Confirm username @{username} </button>
            {/if}
          </div>
    </form>
    {/if}
</AuthCheck>
