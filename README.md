**Οδηγίες για το AWGN κανάλι με BPSK διαμόρφωση υλοποιημένο σε VHDL**
___

Βασικά η παρακάτω αναφορά δεν έχει σκοπό να δείξει την αρχη λειτουργία του component LLR channel αλλά το τρόπο και τους χρονισμούς του , ώστε να κατασκευαστεί ένα FMS απλά και γρήγορα.

Αρχικά αυτό το component απαρτίζεται από επιμέρους component. Ο θόρυβος θα πρέπει να μπαίνει σε κάθε bit της κωδικοποιμένης λέξης και η έξοδος θα πρέπει να είναι ένας πίνακας όπου η μια διάσταση είναι ίση με το μήκος της εισόδου και η άλλη διάσταση είνια το πλήθος των bits που εκφράζουν το εν λόγω LLR ενός bit.

Για να μπεί θόρυβος σε κάθε bit της λέξης εισόφου θα πρέπει να γίνει parallel-2-serial και αυτό γίνεται με το component mux το οποίο παίρνει σαν είσοδο την κωδική λέξη και βγάζει 2 bits (llr2bit) αυτής σε κάθε clk. ΤΟ σήμα llr2bit θα μπει σαν είσοδο στο component *mysystem* το οποίο θα παράξει το διαμορφωμένο σύμβολο για κάθε bit με διαμόρφωση BPSK.
Η διαμόρφωση BPSK είνια γνωστό ότι μπορεί να περιγραφεί με την εξίσωση 
BPSK = 1 - 2 X , X είναι ένα bot της κωδικοποιημένης λέξης.

Η σχέση που υπολογίζει τα LLR είναι  2y/σ^{2} ,	y=x+σ 

Οι τιμές των σ και σ^{2} , για διάφορες στάθμες θορύβου έχουν υπολογιστεί μέσω *Matlab* και έχουν εισαχθεί σε ένα LUT (snrlut.vhd), όπου για κάθε στάθμη θορύβου αξάγωνται αυτές οι δύο ποσότητες και χρησιμοποιούνται από το "*mysystem.vhd*".
Για λόγους debugging για 0dB έχει οριστεί το  σ=0 και το σ^{2}=-1,  το οποίο θα παράξει LLR μέγιστης αξιοπιστίας και ο decoder θα βρει την κωδική λέξη χωρίς κανένα λάθος. Αν στην ίδια περίπτωση το σ=1 με αντίστοιχο format (8.32) αναμένεται να γίνει μέτρηση στο 0dB SNR.

Έχοντας το σ=0 και το σ^{2}=-1 θα πρέπει η διαμόρφωση να γίνεται όχι όπως αναφέρεται παραπάνω, αλλα BPSK = 2X - 1.


Το parallel-2-serial γίνεται χρησιμοποιώντας έναν counter του οποίου η έξοδος θα χρησιμοποιηθεί για να επιλεχθεί στο *mux.vhd* σε ποια δυάδα bits θα προστεθεί θόρυβος. Επομένως αν ο counter έχει στην έξοδο του τη τιμή 5, το 10 και το 11 bit της λέξης εισόδου θα επιλεχθεί για να του προστεθεί θόρυβος.
Η αντίστροφη διαδικασία γίνεται και για την έξοδο, όπου κάθε φορά υπολογίζονται τα llr1 και llr2 από το *mysystem*, δηλαδή όταν παραχθεί ένα ζεύγος από llr θα αποθηκευτούν σε έναν register-file, όπου το σήμα enable για κάθε καταχωρητή παράγεταθ από έναν shift-register. O shift-register θα πρέπει να γίνει enable για __ένα clk και μόνο__ καθώς σε κάθε clk πρέπει να ενεργοποιείται μόνο ένας καταχωρητής του register-file. Όταν τελειώσει το shifting και μόνο τότε είναι σωστά τα llr και έτοιμα για το decoder.

Το component “LLR_channel” έχει σαν είσοδο όπως αναφέρθηκε παραπάνω την λέξη εισόδου, llr_count_en	: ενεργοποιεί τον 


- llr_counter  για το parallel-2-serial
- dinredy	: να είναι μόνιμα στο ‘1’
- start_shift	: ενεργοποιει το shift, να γίνει ‘1’ για ένα clk μονο
- SNR		: σταθμη θορύβου
- llr_counter	: ο counter για το p-2-s
- shift_reg	: 
- readyout	: όταν γίνει ‘1’ μετά το rst του συστήματος, τοτε μπορεί να βγάλει σωστά LLR

Το πρόβλημα που γεννιέται είναι τότε θα πρέπει να γίνει το llr_count_en =’1’ και πότε το start_shift=’1’, τα οποία σήματα θα πρέπει να ελέγχονται από την FSM.
Όταν  είναι έτοιμη η κωδικοποιημένη λέξη και διαθέσιμη στην είσοδο του καναλιού θα πρέπει να γίνει στο επόμενο clk  το llr_count_en =’1’.
Θα πρέπει όταν γίνει ο llr_counter = “7”  θα πρεπει να γίνει το start_shift =’1’ για ένα clk.
Το πιο σημαντικό είναι ότι δεν θα πρέπει ο llr_counter να πάθει overflow ενώ γίνεται shifting. Άρα θα  πρέπει να παραμείνει στην τιμή “FF” (το πόσα F εξαρτάται απο το μήκος της λέξης), μέχρι να είναι διαθέσιμη η επόμενη κωδικοποιημενη λέξη ή να μην χρειάζεστε άλλο τα LLR.

Το πως θα γίνει η παραπάνω διαδικασία είναι στην δικιά σας ευχέρια για το γράψιμο της FSM.
