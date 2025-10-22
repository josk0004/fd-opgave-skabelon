# Reflektion til implementering af et Figma Design

## Udfordringer og successer

Nogle af komponenterne har jeg implementeret fra tidligere opgaver/projekter, som f.eks. `AccordionContainer.astro`, `AccordionItem.astro` og `ScrollingContainer.astro`. For Accordion komponentet var det en udfordring at få det til at fungere med `faq.json` filen, og få den til at indhente og vise dataen.

### Accordion

I `about.astro` tilføjer jeg et id til hver af faq elementerne, da `AccordionItem.astro` benytter sig af det.

```js
const items = faqData.faq.map((item, idx) => ({
  id: `faq-${idx + 1}`,
  question: item.question,
  answer: item.answer,
}));
```

I `AccordionItem.astro`, bliver `answer` splittet op i afsnit ved tomme linjer vha. regex `/\n\s*\n/`.

```js
...answer.split(/\n\s*\n/)...
```

### Scrolling Container

`ScrollingContainer.astro` skulle også have nogle opdateringer for at virke rigtigt og matche figma designet.

I stedet for at brugeren scroller sidelens med musen, skal de nu benytte sig af knapper, og kortet der er i focus skal se anderledes ud.

Det opnåede jeg ved at give hvert af list elementerne et index tal og hved brug af `scrollIntoView`. Når brugeren klikker på højre eller venstre pilene, tæller den op/ned og kortet scroller ind og får tilføjet class'en "focused".

```js
const items = Array.from(list.children);
let focusedIndex = 0;

function updateFocus() {
  items.forEach((item, idx) => {
    item.classList.toggle("focused", idx === focusedIndex);
  });
}

document.querySelector(".arrows .left").parentElement.addEventListener("click", () => {
  focusedIndex = Math.max(0, focusedIndex - 1);
  items[focusedIndex].scrollIntoView({ behavior: "smooth", block: "nearest", inline: "center" });
  updateFocus();
});
```

### Conic-gradient animerede cirkler

Men nok det mest udfordrende komponent var `Experience.astro`, på grund af de animerede cirkler og procent tal.

Til at starte med undersøgte jeg hvordan man lavede de animerede cirkler ved brug af `conic-gradient` og `@property`. Til det fandt jeg siden: https://www.smashingmagazine.com/2023/10/animate-along-path-css/, adapterede det så det matchede figma designet og gjorde så cirklerne animere når de kommer frem på skærmen i stedet for `:hover`.

For at få det til at fungere blev jeg nød til at bruge JavaScript i stedt for CSS.

`@property --p` får en inital value sat til 0%. Når cirklerne bliver vist på skermen opdatere jeg procenten for hver af cirklerne med deres respektive procenter. Jeg undersøger hvornår cirklerne er på skærmen med `IntersectionObserver`.

```js
 circle.style.setProperty("--p", "0%");
      if (indicator) {
        indicator.style.setProperty("--p", "0%");
      }

      const observer = new IntersectionObserver(
        (entries) => {
          entries.forEach((entry) => {
            if (entry.isIntersecting) {
              setTimeout(() => {
                circle.style.setProperty("--p", percentage);
                if (indicator) {
                  indicator.style.setProperty("--p", percentage);
                }
```

Til procent tallet der tæller op i midten af cirklen bruger jeg `performance.now()` og `requestAnimationFrame` til at beregne hvor langt animationen er kommet (`progress` fra 0 til 1).

Jeg anvender en "ease-in-out" kurve, så animationen starter langsomt, accelererer i midten, og bremser op til sidst, ligesom selve cirklen gør med CSS.

For hvert frame beregnes den aktuelle værdi (`currentValue`) baseret på progress, og tallet opdateres i DOM'en via `textContent`.

Når `progress` når 1, stopper animationen og sætter det endelige tal.

```js
const duration = 2000;
const startTime = performance.now();

const animateNumber = (currentTime: number) => {
  const elapsed = currentTime - startTime;
  const progress = Math.min(elapsed / duration, 1);

  const easeInOutProgress = progress < 0.5 ? 2 * progress * progress : 1 - Math.pow(-2 * progress + 2, 2) / 2;

  const currentValue = Math.floor(easeInOutProgress * value);
  if (numberDisplay) {
    numberDisplay.textContent = `${currentValue}%`;
  }

  if (progress < 1) {
    requestAnimationFrame(animateNumber);
  } else if (numberDisplay) {
    numberDisplay.textContent = `${value}%`;
  }
};
```

## CSS organisering

CSS'en er organiseret i et hierarki ved hjælp af CSS Cascade Layers importeret i `main.css`:

```css
@import url("./reset.css") layer(reset);
@import url("./global.css") layer(global);
@import url("./overrides.css") layer(overrides);
```

**Global CSS** indeholder:

- CSS custom properties (farver, typography) i `:root` som f.eks. `--color-primary-dark: #181818` og `--font-body: 'Lato', sans-serif`

Optimalt ville man nok have defineret mere af CSS'en i `global.css` for at undgå repetition i de individuelle komponenter, og lavet et bedre heiraki for `h` elementerne blandt andet.

#

**Komponent-specifik CSS** placeres i `<style>` tags i hver `.astro` fil og wrapper i `@layer components`

Jeg benyttede mig meget af CSS nesting i komponenterne for at have et bedre overblik. `@media` queries nestede jeg også, så det var lettere at se hvad de hørte til. I forbindelse med nesting brugte jeg også selectors som `:not`, `:first-child`, `+` og `> *`.
