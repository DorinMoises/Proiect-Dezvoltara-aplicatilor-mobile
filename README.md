# Proiect-Dezvoltara-aplicatilor-mobile
titlul spune tot

aplicatia face 3 lucruri:

1->Efectuează note prin utilizarea voice-to-text sau de la tastatură, tradițional.
2->Salvează notele la spațiul de stocare local.
3->Afișează toate notele și le oferă posibilitatea de a le asculta prin intermediul sintezei Speech.

1.Speech to Text:

API-ul Web Speech este separat de două interfețe complet independente. Avem SpeechRecognition pentru înțelegerea vocii umane și transformarea acesteia în text (Speech -> Text) și SpeechSynthesis pentru citirea șirurilor cu voce tare într-o voce generată de calculator (Text -> Speech).

Interfața de recunoaștere a vorbirii este surprinzător de precisă pentru o caracteristică gratuită a browserului. A recunoscut corect toate vorbele mele și a știut ce cuvinte merg împreună pentru a forma fraze care au sens. De asemenea, vă permite să dictați caractere speciale precum opriri complete, semne de întrebare și linii noi.

Primul lucru pe care trebuie să-l facem este să verificăm dacă utilizatorul are acces la API și să afișeze un mesaj de eroare adecvat. Din nefericire, API-ul de tip speech-to-text este acceptat numai în Chrome și Firefox (cu un pavilion), astfel încât o mulțime de oameni vor vedea probabil acel mesaj.

try {

  var SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
  
  var recognition = new SpeechRecognition();
  
}

catch(e) {

  console.error(e);
  
  $('.no-browser-support').show();
  
  $('.app').hide();
  
}

Variabila de recunoaștere ne va oferi acces la toate metodele și proprietățile API. Există mai multe opțiuni disponibile, dar vom seta doar recunoașterea.continuă la adevărat. Acest lucru va permite utilizatorilor să vorbească cu pauze mai lungi între cuvinte și fraze.
Înainte de a putea folosi recunoașterea vocii, trebuie să organizăm și o serie de operatori de evenimente. Majoritatea dintre aceștia ascultă pur și simplu schimbări în starea de recunoaștere:

recognition.onstart = function() { 

  instructions.text('Voice recognition activated. Try speaking into the microphone.');
  
}

recognition.onspeechend = function() {

  instructions.text('You were quiet for a while so voice recognition turned itself off.');
  
}

recognition.onerror = function(event) {

  if(event.error == 'no-speech') {
  
    instructions.text('No speech was detected. Try again.');  
    
  };
  
}

Există, totuși, un eveniment special, care este foarte important. Se execută de fiecare dată când utilizatorul vorbește un cuvânt sau mai multe cuvinte în succesiune rapidă, oferindu-ne acces la o transcriere text a ceea ce sa spus.Atunci când captem ceva cu ajutorul handlerului onresult îl salvăm într-o variabilă globală și îl afișăm într-o textarea:

recognition.onresult = function(event) {

  // event is a SpeechRecognitionEvent object.
  
  // It holds all the lines we have captured so far. 
  
  // We only need the current one.
  
  var current = event.resultIndex;
  
  // Get a transcript of what was said.
  
  var transcript = event.results[current][0].transcript;
  
  // Add the current transcript to the contents of our Note.
  
  noteContent += transcript;
  
  noteTextarea.val(noteContent);
  
}

Codul de mai sus este ușor simplificat. Există o eroare foarte ciudată pe dispozitive Android care face ca totul să fie repetat de două ori. Nu există încă o soluție oficială, dar am reușit să rezolv(am cautat online) problema fără efecte secundare evidente. Cu acest bug în minte, codul arată astfel:

var mobileRepeatBug = (current == 1 && transcript == event.results[0][0].transcript);

if(!mobileRepeatBug) {

  noteContent += transcript;
  
  noteTextarea.val(noteContent);
  
}

După ce am instalat totul, putem începe să folosim funcția de recunoaștere vocală a browserului. Pentru a începe, pur și simplu apelați metoda start ():

$('#start-record-btn').on('click', function(e) {

  recognition.start();
  
});

Acest lucru va determina utilizatorii să dea permisiunea. Dacă acest lucru este acordat, microfonul dispozitivului va fi activat.
Browserul va asculta un timp și fiecare frază sau cuvânt recunoscut va fi transcris. API-ul va înceta să asculte automat după câteva secunde de tăcere sau când se oprește manual.

$('#pause-record-btn').on('click', function(e) {

  recognition.stop();
  
});

Cu aceasta, porțiunea de vorbire-text a aplicației noastre este completă!

2.Text to Speech

Speech Synthesys este de fapt foarte ușor. API-ul este accesibil prin obiectul speechSynthesis și există câteva metode de redare, întrerupere și alte chestii legate de sunet. De asemenea, are câteva opțiuni reci care schimbă pitch-ul, rata și chiar și vocea cititorului.

Tot ce vom avea de fapt nevoie pentru demonstrația noastră este metoda speak (). Se așteaptă un argument, o instanță a clasei SpeechSynthesisUtterance numită frumos.

Iată întregul cod necesar pentru a citi un șir:

function readOutLoud(message) {

  var speech = new SpeechSynthesisUtterance();

  // Set the text and voice attributes.
  
  speech.text = message;
  
  speech.volume = 1;
  
  speech.rate = 1;
  
  speech.pitch = 1;

  window.speechSynthesis.speak(speech);
  
}

Atunci când această funcție este chemată, o voce robot va citi șirul dat, făcând cea mai bună impresie umană.
API-urile de sinteză a vorbirii și de recunoaștere a vorbirii funcționează destul de bine și se ocupă cu ușurință de diferite limbi și accente. Din păcate, pentru moment, au limitate de asistență pentru browser, ceea ce îngreunează utilizarea lor în producție. Dacă aveți nevoie de o formă mai fiabilă de recunoaștere a vorbirii, aruncați o privire la aceste API :

https://cloud.google.com/speech/

https://azure.microsoft.com/en-us/services/cognitive-services/speech/

https://cmusphinx.github.io/

https://api.ai/
