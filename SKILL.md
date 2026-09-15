---
name: fact-check
description: |
  Verifică un document față de sursele lui și produce o „copie ancorată”: o pagină HTML în care
  fiecare afirmație verificată poartă bule colorate de citare care deschid pasajul verbatim din
  sursă, cu vizualizatoare care sar la textul citat și îl evidențiază.
  Folosește când utilizatorul spune „verifică documentul ăsta”, „fact-check”, „ancorează documentul
  în surse”, „verifică comunicatul înainte să-l trimitem”, „verifică materialul față de raport”,
  „verifică citatele din transcript”.
license: MIT
compatibility: claude-code
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - WebSearch
  - WebFetch
---

# Fact-check → copie ancorată (ediția de curs)

Această ediție nu are nevoie de nimic instalat: fără Python, fără Node, fără `pandoc`,
`pdftotext` sau `curl`. Singurul lucru necesar este un browser. Verificarea structurală
care în ediția completă rula într-un script Python rulează acum **în pagina însăși**,
la deschidere.

Livrabilul **nu este un răspuns în chat**. Este un folder pe care utilizatorul îl deschide
în browser: o copie fidelă a documentului verificat, în care fiecare afirmație verificată
are bule de citare pe care se poate da click.

```
<folderul documentului>/
└── verificat/<numedoc>/
    ├── verificare.html   ← COPIAT identic din skill, niciodată editat
    └── date.js           ← SINGURUL fișier pe care îl scrii tu
```

**Scrii exact un fișier: `date.js`.** Numerele de linie, marcajele din surse, numerotarea
bulelor, culorile, totalurile și linkurile sunt derivate de motor la deschiderea paginii.
Asta e intenționat: **un localizator pe care nu îl poți scrie este un localizator pe care
nu îl poți inventa.**

> **Compatibilitate.** Motorul (`assets/verificare.html`) e complet offline: nicio resursă externă, nicio
> rețea. Datele se încarcă printr-un `<script src="./date.js">`, iar confruntarea din bannerul de sus folosește
> FileReader local. Pagina se deschide în orice browser, fără nimic instalat.
>
> Skill-ul pornește cu `/fact-check` sau e recunoscut din descriere; e în `.claude/skills/` al folderului
> deschis, deci nu are nevoie de nicio instalare.
>
> Accesul la internet pentru recursul la web depinde de setările organizației — dacă e blocat, se lucrează doar
> pe sursele locale, iar afirmațiile neacoperite rămân `notfound` cu nota corespunzătoare.

---

## Pasul 1 — Identifică intrările

- **Documentul de verificat**: numit în cerere, altfel întreabă. Un singur document pe rulare.
- **Sursele**: fiecare fișier relevant din același folder, plus orice URL dat de utilizator.
  Enumeră-le înapoi utilizatorului dacă există ambiguitate.
- **Materialul derivat** (un explainer, un comunicat anterior, rezumatul altcuiva) **nu este
  sursă primară**. Folosește-l doar dacă nimic primar nu acoperă afirmația, și spune asta în
  `note`-ul acelei citări. Pentru o companie listată, sursa primară e raportul depus la bursă,
  nu articolul care îl rezumă.
- **URL-urile și DOI-urile pe care documentul însuși le citează sunt surse.** Un document care
  citează ceva pretinde implicit că acel ceva îl susține — deci verifică-l.

## Pasul 2 — Citește sursele

Nu ai nevoie de niciun convertor extern. Folosește uneltele tale:

| Sursa | Cum o citești |
|---|---|
| `.md` / `.txt` | `Read` |
| `.pdf` cu text | `Read` — citește PDF-uri nativ, pagină cu pagină (`pages: "1-5"`) |
| `.docx` | `Read`; dacă nu merge, spune-i utilizatorului și cere un export `.txt`/`.md` |
| `http(s)://` | `WebFetch` pe URL |
| **`.pdf` scanat / poză / document fără strat de text** | vezi secțiunea următoare |

Citește sursele **integral**, nu doar rezultate de `Grep`. Textul pe care îl pui în `date.js`
trebuie să fie textul pe care l-ai citit efectiv, transcris exact.

### Surse scanate — transcriere cu subagent

Un PDF scanat, o poză a unui document sau un fax vechi nu au strat de text: `Read` îți dă
imaginea, nu cuvintele. La scara unui caz (2–20 de pagini), nu ai nevoie de un serviciu extern
de parsare — pui un **subagent** să transcrie.

**Cum:**

1. Pornește un subagent cu **o singură sarcină**: transcrie paginile, verbatim.
   Unde poți alege modelul, **Sonnet** e potrivirea corectă — e o sarcină bine delimitată de
   execuție, nu de judecată. Judecata rămâne la modelul principal, în Pasul 3.
2. Instrucțiunea pentru subagent, cuvânt cu cuvânt în spirit: *transcrie exact ce vezi. Nu
   corecta greșeli, nu reordona, nu completa, nu rezuma, nu moderniza ortografia. Păstrează
   numerele exact cum sunt tipărite. Ce nu se citește marchează cu* `[ILIZIBIL]`. *Marchează
   antetele de tabel și păstrează structura tabelului.*
3. Subagentul scrie rezultatul **într-un fișier pe disc**, lângă scan:
   `surse/<nume>-transcriere.txt`. Nu ți-l returnează doar în conversație.
4. În `date.js`, sursa este **fișierul de transcriere**, nu scanul: `kind: "file"`,
   `name: "<nume>-transcriere.txt"`, iar `note` spune explicit
   *„transcriere AI a scanului `<nume>.pdf`; pasajele citate au fost confruntate cu imaginea”*.

**De ce subagent și nu în conversația principală:** transcrierea integrală e lungă și ți-ar
umple contextul cu text pe care oricum îl citești din fișier mai târziu.

**Regula de onestitate, care nu se negociază.** Motorul verifică doar că fiecare fragment există
în textul sursei din `date.js`. Dacă acel text e o transcriere AI, bannerul verde dovedește
**consistență internă, nu fidelitate față de scan**. Prin urmare:

- fiecare fragment citat dintr-o sursă transcrisă se confruntă **de un om, pe imagine**, înainte
  de predare — doar pasajele citate, nu tot documentul;
- o afirmație cu risc mare (cifră, citat, superlativ, obligație contractuală) nu rămâne susținută
  **exclusiv** de un scan transcris;
- dacă o cifră e ambiguă în imagine, verdictul nu e `confirmed`. Marchează `[ILIZIBIL]` și cere
  documentul în format digital.

**Când un serviciu specializat de parsare rămâne mai bun:** zeci sau sute de documente, tabele
complexe la scară, sau când ai nevoie de o parsare repetabilă, identică la fiecare rulare. Pentru
un teanc, acela e specialistul. Pentru câteva pagini, subagentul face aceeași treabă și nu adaugă
încă un furnizor prin care trec documentele clientului.

## Pasul 3 — Citește și fă munca de judecată

Asta e singura parte care îți aparține.

Marchează fiecare: cifră, procent, cantitate, dată, DOI, număr de contract; entitate numită
(persoană, instituție, publicație, proiect) **și titlul/rolul ei**; citat direct **și atribuirea
lui**; afirmație de superlativ sau de precedență („prima platformă din România”, „liderul
pieței”); afirmație cauzală sau de mecanism; afirmație despre disponibilitate, aplicabilitate
sau limitări.

Tipic 12–20 de afirmații pentru un comunicat, mai multe pentru un articol lung. Tu decizi
cuvintele exacte ale fiecărei afirmații — intervalul ales trebuie să fie **cel mai scurt text pe
care un cititor ar trebui să îl schimbe dacă afirmația ar fi greșită**.

### Reguli de onestitate — ele guvernează judecata

Pentru fiecare afirmație găsește pasajul din surse și notează: **fragment verbatim**, **verdict**,
opțional o **notă** de un rând.

Verdicte:

- `confirmed` — sursa spune asta.
- `partial` — cifrele se potrivesc dar încadrarea diferă, afirmația e mai tare decât sursa,
  s-a pierdut context, sau sursa nuanțează unde documentul nu o face.
- `contradicted` — sursa spune ceva incompatibil.
- `notfound` — **doar ca ultimă soluție.** Înseamnă: nu e în sursele date **și** nu e verificabil
  nici pe web. Lasă `excerpt` gol; `note` trebuie să spună ce surse locale s-au căutat cu ce
  termeni **și** ce căutări web s-au încercat.

### Recurs la web — înainte să scrii vreodată `notfound`

O afirmație neacoperită de sursele locale nu trebuie să se înfunde într-o bulă gri. Caută pe web
(`WebSearch` / `WebFetch`) o pagină autoritară. Dacă un pasaj verbatim o susține sau o contrazice,
citeaz-o ca sursă web normală, cu verdictul potrivit.

- **Sursele locale câștigă mereu.** Recursul la web e pentru afirmații pe care nicio sursă locală
  nu le acoperă — sau, rar, ca a *doua* citare care adaugă confirmare cu adevărat independentă.
  Nu împrăștia bule verzi de web peste afirmații deja confirmate local.
- **Autoritate.** Preferă documentul depus de companie (raport la bursă, raport anual auditat),
  paginile instituționale, registrele și agențiile oficiale, publicațiile de referință. Fără ferme
  de conținut, fără bloguri SEO, fără agregatoare. Dacă cel mai bun lucru găsit e de mâna a doua,
  spune-o în `note`.
- **Recursul la web se aplică faptelor despre lume, niciodată atribuirii citatelor.** Nicio pagină
  web nu poate stabili ce a spus o persoană într-un interviu — doar înregistrarea sau transcriptul
  pot. Dacă un citat lipsește din transcript, rămâne `notfound` sau `contradicted`, iar nota spune
  că ancorarea web nu se aplică citatelor.
- **Fă proveniența evidentă.** Pentru o sursă web, `name` este domeniul (`bvb.ro`, `zf.ro`) iar
  `note` trebuie să spună că afirmația a fost ancorată de pe web, nu din sursele furnizate.

Reguli dure:

- **Nu inventa niciodată un fragment.** Fiecare `excerpt` trebuie să fie text pe care l-ai găsit
  efectiv în sursă, copiat, nu parafrazat. Dacă nu îl găsești, verdictul este `notfound`.
  Motorul refuză să afișeze un fragment care nu există în textul sursei — apare o eroare roșie.
- **Verifică fiecare citat direct cuvânt cu cuvânt față de transcript.** Un citat care nu a fost
  rostit, sau a fost rostit de altcineva, sau căruia i s-a adăugat o a doua propoziție, este o
  *constatare* (`contradicted` / `partial`), nu o potrivire. Dacă transcriptul e ASR și citatul e
  o variantă curățată a unui pasaj bruiat, asta e `confirmed` — dar spune-o în `note`.
- **O afirmație poate purta mai multe citări cu verdicte diferite.** Ăsta e rostul formatului: o
  bulă roșie către pasajul care contrazice, lângă o bulă verde către pasajul cu cifra corectă,
  îi arată cititorului conflictul dintr-o privire.
- **Verifică documentul față de el însuși.** Titluri, nume sau cifre inconsecvente între paragrafe
  sunt constatări — încadrează fiecare apariție ca afirmație separată, ca să fie vizibil conflictul.
- Contextul pierdut (un procent fără numitorul lui, un profit fără sursa lui) este `partial`,
  nu `confirmed`.

**Nu scrii niciun număr de linie și nicio pagină.** Motorul le derivă din fragment.

## Pasul 4 — Construiește folderul

Creează folderul și copiază motorul. Nu îl edita și nu îl rescrie de mână:

```bash
mkdir -p "<folder>/verificat/<numedoc>"
cp "<cale către skill>/assets/verificare.html" "<folder>/verificat/<numedoc>/"
```

Dacă `cp` nu e disponibil sau e blocat de permisiuni, citește `assets/verificare.html` cu
`Read` și scrie-l identic cu `Write`. Rezultatul e același; nu schimba nimic în conținut.

## Pasul 5 — Scrie `date.js`

Fișierul conține **exact o instrucțiune**: `window.FACTCHECK = <JSON valid>;`
Folosește ghilimele duble și `\n` pentru rândurile din texte — adică JSON curat, nu template literals.

```js
window.FACTCHECK = {
  "lang": "ro",
  "kicker": "Copie verificată · comunicat de presă",
  "title": "Titlul documentului",
  "meta": "Document verificat: … · Surse: … · Verificare: <data>",
  "footer": "notă de proveniență, poate conține HTML inline",

  "document": {
    "name": "comunicat-draft.md",
    "text": "textul brut INTEGRAL al documentului, exact ca în fișier"
  },

  "sources": [
    {
      "id": "raport_s1",
      "name": "raport-semestrial.pdf",
      "kind": "file",
      "note": "un rând afișat în antetul vizualizatorului",
      "text": "textul brut INTEGRAL al sursei"
    },
    {
      "id": "bvb",
      "name": "bvb.ro",
      "kind": "web",
      "url": "https://bvb.ro/...",
      "title": "Titlul paginii",
      "note": "pagină autoritară pentru o afirmație neacoperită local",
      "text": "textul extras cu WebFetch"
    }
  ],

  "claims": [
    {"text": "cuvintele exacte din document", "cites": ["c1", "c2"], "occurrence": 1}
  ],

  "citations": [
    {
      "id": "c1",
      "source": "raport_s1",
      "verdict": "confirmed",
      "excerpt": "cuvintele verbatim din sursă",
      "excerptOccurrence": 1,
      "hint": "Rezultate consolidate",
      "note": "un rând, opțional"
    }
  ]
};
```

Note despre câmpuri:

- `document.text` și `sources[].text` sunt **textul brut integral**, transcris exact. Motorul îl
  folosește ca să redeseneze documentul și ca să regăsească fiecare fragment. Nu prescurta, nu
  rezuma, nu „curăța” textul — dacă îl modifici, participantul va vedea asta când confruntă
  fișierele originale.
- `claims[].text` se caută în textul brut al documentului (markdown cu tot), nu în HTML-ul redat.
  Spațiile albe se colapsează la căutare, deci o afirmație poate trece peste o întrerupere de rând.
  `occurrence` (de la 1) e necesar doar când aceleași cuvinte apar de mai multe ori — exact așa
  faci vizibilă o auto-contradicție: două afirmații, același text, `occurrence` diferit, citări diferite.
- O afirmație trebuie să încapă **într-un singur paragraf**. Dacă trece peste un rând gol, motorul
  semnalează eroare; împarte-o în două.
- `citations[].source` trebuie să fie un `id` din `sources`.
- `excerpt` se caută la fel de flexibil la spații, deci un pasaj care se întinde pe mai multe rânduri
  se scrie ca un singur șir. `excerptOccurrence` dezambiguizează repetițiile.
- `hint` este eticheta umană de după numărul de linie („Rezultate consolidate”, „Nota 7”,
  „Concluzii”). **Niciodată numere acolo** — ele sunt derivate.
- Citările `notfound`: `excerpt` gol, un `note` care spune ce s-a căutat local **și** pe web,
  fără `source`.
- `ui` este opțional; etichetele implicite sunt deja în română. Suprascrie-l doar pentru altă limbă.

### Ce derivă motorul, ca să nu scrii tu

numerele de linie ale fiecărui fragment · vizualizatorul fiecărei surse, cu rânduri numerotate și
`<mark>` pe pasajul citat · numerotarea, culorile și sublinierile bulelor · banda de sumar și lista
constatărilor · linkurile text-fragment (`#:~:text=…`) către pagina web live, cu cel mai scurt prefix
unic de 5–12 cuvinte.

## Pasul 6 — Verifică deschizând pagina

Deschide `verificare.html` în browser. **Bannerul din capul paginii este rezultatul verificării**:

- **roșu** — probleme structurale. Motorul le enumeră pe toate deodată: o afirmație al cărei text
  nu e în document, o afirmație ambiguă fără `occurrence`, un fragment care nu e în sursa lui, o
  citare pe care nu o folosește nimeni, un verdict necunoscut, un `notfound` cu fragment, două
  citări cu același `id`, afirmații care se suprapun. **Repară `date.js` și reîncarcă pagina.**
- **chihlimbariu** — structura e validă, dar textele surselor incluse în pagină nu au fost încă
  confruntate cu fișierele de pe disc.
- **verde** — totul validat, inclusiv confruntarea cu originalele.

Ca să ajungi la verde, trage documentul și fișierele sursă originale în zona de drop din banner.
Browserul le compară cu textele incluse în pagină și marchează fiecare fișier `identic` sau
`DIFERIT`. Nimic nu se trimite nicăieri — comparația e locală.

**Fă tu însuți acest gest înainte de a preda rezultatul.** E singurul lucru care dovedește că
textele pe care le-ai transcris în `date.js` sunt textele reale.

Pentru sursele scanate, în zona de drop se trage **fișierul de transcriere**, nu scanul — și
verde înseamnă atunci „fragmentele există în transcriere”. Fidelitatea transcrierii față de
imagine o confirmă omul, pe pasajele citate. Spune asta explicit la predare.

Dacă bannerul e roșu, **nu preda rezultatul** — repară `date.js` și reîncarcă.

## Pasul 7 — Predă

- Spune calea exactă a fișierului și cum se deschide (dublu-click pe `verificare.html`).
- Rezumă în chat, în limbaj simplu, **doar constatările roșii / chihlimbarii / gri** — ce pretinde
  documentul, ce spune de fapt sursa, ce trebuie schimbat. Fără jargon.
- Dacă ai pornit să verifici o problemă așteptată și s-a dovedit în regulă, spune explicit asta.
- Dacă vreo sursă a fost transcrisă dintr-un scan, spune care, și ce pasaje au fost confruntate
  cu imaginea.
- Cele două fișiere trebuie să rămână împreună în același folder. `verificare.html` singur, mutat
  în altă parte, va afișa „Datele lipsesc”.
