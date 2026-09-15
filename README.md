# fact-check

Skill pentru [Claude Code](https://claude.com/claude-code) care verifică un document față de sursele
lui și produce o „copie ancorată”: o pagină HTML în care fiecare afirmație verificată are o bulă
de citare colorată, ce deschide pasajul exact din sursă.

Funcționează complet offline — nu are nevoie de Python, Node, pandoc sau alte unelte instalate.
Singurul lucru necesar e un browser.

## Instalare

Un „skill” Claude Code este pur și simplu un folder cu instrucțiuni, pe care Claude Code îl citește
automat dacă e pus la locul potrivit. Nu se instalează ca un program — doar se copiază folderul.

**Pasul 1 — descarcă skill-ul**

```bash
git clone https://github.com/madascicom/fact-check.git
```

Sau descarcă arhiva ZIP de pe pagina repo-ului (butonul verde **Code → Download ZIP**) și dezarhiveaz-o.

**Pasul 2 — pune-l în locul potrivit**

Claude Code caută skill-uri într-un folder ascuns numit `.claude/skills/`. Poți să-l adaugi în două locuri,
în funcție de cât de larg vrei să fie disponibil:

- **Pentru un singur proiect** — copiază folderul `fact-check` în `<proiectul-tău>/.claude/skills/fact-check`
- **Pentru toate proiectele tale** — copiază folderul `fact-check` în `~/.claude/skills/fact-check`
  (pe Windows: `C:\Users\<numele-tău>\.claude\skills\fact-check`)

Structura finală trebuie să arate așa:

```
.claude/skills/fact-check/
├── SKILL.md
└── assets/
    └── verificare.html
```

**Pasul 3 — verifică**

Deschide (sau repornește) Claude Code în acel folder și scrie `/fact-check`, sau pur și simplu cere
„verifică documentul ăsta față de surse” — Claude Code recunoaște skill-ul automat din descrierea lui.

## Cum se folosește

1. Ai un document (comunicat, articol, transcript) și sursele lui (rapoarte, PDF-uri, pagini web) în
   același folder.
2. Ceri lui Claude Code să-l verifice: *„fact-check documentul ăsta”* sau *„verifică comunicatul înainte
   să-l trimitem”*.
3. Claude Code citește sursele, marchează fiecare afirmație verificabilă și produce un folder nou:
   `verificat/<numedoc>/` cu `verificare.html` (motorul, copiat identic) și `date.js` (datele verificării).
4. Deschizi `verificare.html` cu dublu-click — se deschide în browser, fără nimic de instalat.
   Un banner colorat (roșu/chihlimbariu/verde) arată dacă verificarea e completă și dacă textele
   confruntate corespund cu fișierele originale.

Detalii complete despre reguli, verdicte și formatul datelor sunt în [`SKILL.md`](SKILL.md).

## Licență

MIT
