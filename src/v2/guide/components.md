---
title: Komponenty
type: guide
order: 11
---

## Czym są komponenty?

Komponenty są jedną z najpotężniejszych cech Vue. Pozwalają na rozszeżenie podstawowego html i enkapsulacje go do ponownego uzycia. Na wysokim poziomie, komponenty są personalizowanymi elementami, do których kompilator Vue dodaje dynamikę. W niektórych przypadkach moga wystepować jako natywny HTML rozszeżony o atrybut `is`.

Każdy komponent jest jednocześnie instancja Vue, więc przyjmuje taki sam obiekt z opcjami (poza kilkoma specjanymi opcjami) i oferuje takie same uchwyty cyli życia.

## Korzystanie z komponentów

### Rejestracja globalna

W poprzednich sekcjach tworzyliśmy nowe instacje Vue za pomocą kodu:

``` js
new Vue({
  el: '#jakis-element',
  // opcje
})
```

Aby zrejestrować komponent globalnie, użyj: `Vue.component(tagName, options)`.
Przykład:

``` js
Vue.component('moj-komponent', {
  // opcje
})
```

<p class="tip">Zauważ, że Vue nie wymusza stosowania reguł [W3C](https://www.w3.org/TR/custom-elements/#concepts) dla nazw tagów użytkownika (wszytsko małymi literami, muszą zawierać dywiz) ale przestrzeganie tej konwencji jest dora praktyką.</p>

Zarejestrownay komponent może być uzyty w szablonie instancji jako tag użytkownika `<moj-komponent></moj-komponent>`. Upewnij się, że komponent jest Zarejestrownay **przed** utworzeniem głównej instancji Vue. Poniżej przykład:

``` html
<div id="example">
  <moj-komponent></moj-komponent>
</div>
```

``` js
// rejestracja
Vue.component('moj-komponent', {
  template: '<div>Komponent użytkownika!</div>'
})

// tworzenie głównej instacji
new Vue({
  el: '#example'
})
```

Wyrenderuje:

``` html
<div id="example">
  <div>Komponent użytkownika!</div>
</div>
```

{% raw %}
<div id="example" class="demo">
  <moj-komponent></moj-komponent>
</div>
<script>
Vue.component('moj-komponent', {
  template: '<div>Komponent użytkownika!</div>'
})
new Vue({ el: '#example' })
</script>
{% endraw %}

### Rejestracja lokalna

Nie musisz rejestrować każdego komponentu globalnie. Możesz utworzyć komponent dostępny tylko w zasięgu innej instancji/komponentu, rejestrując go opcją `components` w instancji:

``` js
var potomek = {
  template: '<div>Komponent użytkownika!</div>'
}

new Vue({
  // ...
  components: {
    // <moj-komponent> będzie dostępny tylko w szablonie elementu nadrzędnego
    'moj-komponent': Child
  }
})
```

Ta sama enkapsulacja dotyczy innych zarejestrowanych funkcjonalności Vue, takich jak dyrektywy.

### Parsowanie szablonów DOM

Korzystając z DOM jako szablonu (np: używając opcji `el` do osadzenia elementu z istniejącą zawartością), napotkasz pewne ograniczenia wynikające z działania HTMLa, ponieważ Vue może podbrać zawartość szablonu **po** parsowaniu i normalizacji przez przeglądarkę. Dzieje się tak dlatego, że elementy jak `<ul>`, `<ol>`, `<table>` i `<select>` mogą się pojawić jedynie wewątrz innych elementów.

Doprowadzi to do problemów podczas używania niestandardowych komponentów z elementami, które mają takie ograniczenia, na przykład:

``` html
<table>
  <moj-row>...</moj-row>
</table>
```

Komponent `<my-row>` zostanie potraktowany jako nieprawidłowa zawartość, to może generować błędy podczas renderowania. Obejściem jest zastosowanie atrybutu `is`:

``` html
<table>
  <tr is="moj-row"></tr>
</table>
```

**Te ograniczenia nie wystepują jeżeli wykorzystujesz szablony łańcuchowe z wymienionych źródeł:**

- `<script type="text/x-template">`
- szablon osadzony lokalnie jako łańcuch JavaScript
- komponenty `.vue`

W związku z tym należy korzystać z szablonów łancuchowych, zawsze kiedy to jest mozliwe.

### `data` musi być funkcją

Większość opcji zdefiniowana w konstruktorze Vue może być wykorzystana w komponencie, z jednym zastrzeżeniem: `data` musi być funkcją. Jeżeli użyjesz kod:

``` js
Vue.component('moj-component', {
  template: '<span>{{ komunikat }}</span>',
  data: {
    komunikat: 'Cześć!'
  }
})
```

Vue zatrzyma się i wyświetli w konsoli komunikat: `data` must be a function for component instances (dla instancji komponentu `data` musi byc funkcją). Żeby lepiej zrozumieć zasady spróbujmy nieco oszukać:

``` html
<div id="example-2">
  <prosty-licznik></prosty-licznik>
  <prosty-licznik></prosty-licznik>
  <prosty-licznik></prosty-licznik>
</div>
```

``` js
var data = { licznik: 0 }

Vue.component('prosty-licznik', {
  template: '<button v-on:click="licznik += 1">{{ licznik }}</button>',
  // data technicznie jest funkcją, więc Vue
  // nie wyświetli błędu i zwróci odwołanie do obiektu
  // dla każdej instancji
  data: function () {
    return data
  }
})

new Vue({
  el: '#example-2'
})
```

{% raw %}
<div id="example-2" class="demo">
  <prosty-licznik></prosty-licznik>
  <prosty-licznik></prosty-licznik>
  <prosty-licznik></prosty-licznik>
</div>
<script>
var data = { licznik: 0 }
Vue.component('prosty-licznik', {
  template: '<button v-on:click="licznik += 1">{{ licznik }}</button>',
  data: function () {
    return data
  }
})
new Vue({
  el: '#example-2'
})
</script>
{% endraw %}

Jeżeli każda z instancji komponentu ma ten sam obiekt `data`, inkrementacja jednego licznika inkrementuje wszystkie!
Since all three component instances share the same `data` object, incrementing one counter increments them all! Ouch. Naprawmy to, zwracając nowy obiekt `data`:

``` js
data: function () {
  return {
    licznik: 0
  }
}
```

Teraz każdy licznik będzie miał własny stan:

{% raw %}
<div id="example-2-5" class="demo">
  <moj-komponent></moj-komponent>
  <moj-komponent></moj-komponent>
  <moj-komponent></moj-komponent>
</div>
<script>
Vue.component('moj-komponent', {
  template: '<button v-on:click="licznik += 1">{{ licznik }}</button>',
  data: function () {
    return {
      licznik: 0
    }
  }
})
new Vue({
  el: '#example-2-5'
})
</script>
{% endraw %}

### Komponowanie komponentów

Komponenty powinny być używane razem, zwykle w relacji rodzic - dziecko: komponent A może użyć komponentu B w jego własnym szablonie. Nieunikniona jest komunikacja między nimi: rodzic może oczekiwać danych od dziecka, a dziecko może informować rodzica co się z nim dzieje. Bardzo ważne jest żeby zachować ich rozdzielność, jeżeli to możliwe z wyraźnie zdefiniowanym interfejsem. Daje to pewność, że każdy z komponentów może być napisany utrzymywany we względnej izolacji co sprawia, że będzie nimi łatwiej zarządzać i potencjalnie będą łatwiejsze do ponownego użycia.

W Vue relacje komponentów rodzic-dziecko można podsumować jako: **props przekazywane w dół, zdarzenia emitowane w górę**. Rodzic przekazuje dane do dziecka za pomocą **props**, a dziecko wysyła wiadomości do rodzica za pomocą **zdarzeń**. Zobaczmy jak to funkcjonuje.

<p style="text-align: center;">
  <img style="width: 300px;" src="/images/props-events.png" alt="props down, events up">
</p>

## Właściwość props

### Przekazywanie danych przez właściwość props

Każda instancja komponentu ma swój **izolowany zasięg**. To oznacza, że nie możesz (i nie powinieneś) odwoływać się bezpośrednio do danych rodzica w szablonie komponentu potomnego. Dane moga być przekazywane w dół do komponentu potomnego korzystając z **props**.

Props jest atrybutem użytkownika do przekazywania informacji z nadrzędnego komponentu. Komponent musi mieć jawnie zadeklarowane właściwości, których oczekuje. Deklaruje się je za pomocą opcji [`props`](../api/#props):

``` js
Vue.component('dziecko', {
  // deklaracja właściwości
  props: ['komunikat'],
  // tak jak 'data', 'props' może być uzyte wewnątrz szablonu
  // i jest również dostępny w vm jako this.message
  template: '<span>{{ komunikat }}</span>'
})
```

Wtedy możemy przekazać zwykły ciąg znaków w ten sposób:

``` html
<dziecko komunikat="Cześć!"></dziecko>
```

Result:

{% raw %}
<div id="prop-example-1" class="demo">
  <dziecko komunikat="Cześć!"></dziecko>
</div>
<script>
new Vue({
  el: '#prop-example-1',
  components: {
    dziecko: {
      props: ['komunikat'],
      template: '<span>{{ komunikat }}</span>'
    }
  }
})
</script>
{% endraw %}

### camelCase kontra kebab-case

Atrybuty HTML uwzględniają wielkość liter, więc korzystając z szablonów nie łańcuchowych pisownia nazw właściwości musi być konwertowana z camelCase na kebeb-case (rozdzielone dywizem):

``` js
Vue.component('dziecko', {
  // camelCase w JavaScript
  props: ['mojKomunikat'],
  template: '<span>{{ mojKomunikat }}</span>'
})
```

``` html
<!-- kebab-case w HTML -->
<dziecko moj-komunikat="Cześć!"></dziecko>
```

Ponownie: te ograniczenia nie wystepują jeżeli wykorzystujesz szablony łańcuchowe.

### Dynamiczna właściwość props

Podobnie do bindowania normalnego atrybutu do wyrażenia, również możesz użyć `v-bind` do dynamicznego binowania właściwości do danych w rodzicu. Jeżeli dane zostaną zaktualizowane w rodzicu, zostaną równiez przekazane dziecku:

``` html
<div>
  <input v-model="komunikatRodzica">
  <br>
  <dziecko v-bind:moj-komunikat="komunikatRodzica"></dziecko>
</div>
```

Możesz również korzystać z wersji skróconej `v-bind`:

``` html
<dziecko :moj-komunikat="komunikatRodzica"></dziecko>
```

Wynik:

{% raw %}
<div id="demo-2" class="demo">
  <input v-model="komunikatRodzica">
  <br>
  <dziecko v-bind:moj-komunikat="komunikatRodzica"></dziecko>
</div>
<script>
new Vue({
  el: '#demo-2',
  data: {
    komunikatRodzica: 'Komunikat od rodzica'
  },
  components: {
    dziecko: {
      props: ['mojKomunikat'],
      template: '<span>{{mojKomunikat}}</span>'
    }
  }
})
</script>
{% endraw %}

Jeżeli chcesz przekazać wszystkie właściwości w obiekcie jako właściwość props, możesz uzyć `v-bind` bez argumentu (`v-bind` zamiast `v-bind:nazwa-wlasciwosci`). Na przykład obiekt `todo`:

``` js
todo: {
  tekst: 'Nauczyć się Vue',
  wykonane: false
}
```

Wtedy:

``` html
<todo-item v-bind="todo"></todo-item>
```

będzie ekwiwalentem:

``` html
<todo-item
  v-bind:tekst="todo.tekst"
  v-bind:is-complete="todo.wykonane"
></todo-item>
```

### Literał a dynamika

Czestym błędem początkujacych jest próba ustawienia wartości za pomoca składni literału:

``` html
<!-- ma ustawić właściwość na "1" -->
<comp jakis-prop="1"></comp>
```

Ponieważ jest to literał, przyjmuje warość '"1"' jako łańcuch znaków. Jeżeli chcemy przkazać liczbę, musimy skorzystać z `v-bind`. Kod powinien wyglądać nastepująco:

``` html
<!-- ustawia wartość liczbową 1 -->
<comp v-bind:jakis-prop="1"></comp>
```

### Jednokierunkowy przeplyw dancyh

Wszystkie bindowanego właściwości props tworzą połączenie do **jednokierunkowego przekazywania w dół** pomiędzy dzieckiem, a rodzicem: gdy właściwość w rodzicu zostanie zaktualizowana, zostanie przekazana dziecku ale owrtonie już nie. To zapobiega przypadkowym zmianom stanów komponentów nadrzednych przez potomne, co by sprawiło, że kod aplikacji byłby trudny do zrozumienia.

Pondato każdorazowa aktualizacja komponentu nadrzędnego powoduje odświerzenie wszystkich właściwości props elemendu podrzędnego, do ich ostatnich wartości. To oznacza, że **nie** powinieneś próbować zmieniać właściwości prop wewnątrz komponentu potomnego. Jeżeli to zrobisz Vue ostrzeże Cię komunikatem w konsoli.

Zwykle są dwa powody do zmiany:

1. Prop jest używany do przekazania wartości początkowej; komponent potomny chce później użyć go jako właściwości danych lokalnych.

2. Prop jest przekazywany jako surowa wartość, która musi zostać przekształcona.

Własciwymi rozwiązaniami powyższych problemów są:

1. Wartość początkową właściwości lokalnej zdefiniuj jako wartośc pobieraną z props:

  ``` js
  props: ['initialCounter'],
  data: function () {
    return { counter: this.initialCounter }
  }
  ```

2. Zdefinuj właściwośc wyliczoną, tak aby była wyliczana z props:


  ``` js
  props: ['size'],
  computed: {
    normalizedSize: function () {
      return this.size.trim().toLowerCase()
    }
  }
  ```

<p class="tip">Zauważ, że obiekty i tablice w JavaScript są przekazywane przez odniesienie, więc jeżeli prop jest tablicą lub obiektem, zmiana tego obiektu lub tablicy wewnątrz dziecka **zmieni** stan rodzica.</p>

### Walidacja prop

Jest mozliwość żeby komponent określał wymogi dla przyjmowanych props. Jeżeli nie zostaną spelnione Vue wyswietli ostrzeżenie. Jest to szczególnie przydatne przy pisaniu kompoentów mających być wykorzystywanych przez innych.

Zamiast definiować props w tablicy łańcuchów możesz użyć obiektu z walidacją oczekiwań:

``` js
Vue.component('example', {
  props: {
    // proste sprawdzenie typu danych (`null` oznacza akceptowanie każdego typu)
    propA: Number,
    // wiele dopuszczalnych typów danych
    propB: [String, Number],
    // oczekuje łańcucha
    propC: {
      type: String,
      required: true
    },
    // wartość liczbowa z wartością domyślną
    propD: {
      type: Number,
      default: 100
    },
    // obiekt i tablica domyslnie powinny być zwracane przez funkcję
    propE: {
      type: Object,
      default: function () {
        return { komunikat: 'Cześć' }
      }
    },
    // personalizowana funkcja walidująca
    propF: {
      validator: function (value) {
        return value > 10
      }
    }
  }
})
```

"Typ" może być jednym z następujących natywnych konstruktorów:

- String
- Number
- Boolean
- Function
- Object
- Array
- Symbol

Ponadto, `type` może być funkcją konstruktora użytkownika a asercja zostanie wykonana przy pomocy sprawdzenia `instanceOf`.

Jeżeli prop nie przejdzie walidacji Vue wyświetli ostrzeżenie w konsoli (jeżeli korzystarz z buildu deweloperskiego). Zauważ, że props są walidowane __przed__ utworzeniem instancji, więc w funkcjach `default` lub` validator` właściwości instancji takie jak `data`, `computed` lub `methods` nie będa dostępne.

## Atrybuty Non-Prop

Atrybut non-prop jest atrybutem przekazanym do komponentu nie mającym zdefiniowanego prop docelowego.

Choć do przesyłania informacji komponentowi potomnemu dedykowane są zdefiniowane wcześniej właściwości props, autorzy bibliotek komponentów nie zawsze mogą przewidzieć konteksty, w których mogą być używane ich komponenty. Dlatego komponenty mogą akceptować dowolne atrybuty, które są dodawane do elementu głównego komponentu.

Wyobraź sobie sytuację, że używasz gotowego komponentu `bs-date-input` z wtyczką do bootstrapa, która oczekuje atrybutu `data-3d-date-picker` w `input`. Możemy dodać ten atrybut do naszej instancji komponentu:

``` html
<bs-date-input data-3d-date-picker="true"></bs-date-input>
```

Atrybut `data-3d-date-picker="true"` zostanie automatycznie dodany do głównego elementu `bs-date-input`.

### Wymiana / Scalanie z istniejącymi atrybutami

Wyobraź sobie taki kod w szablonie dla `bs-date-input`:

``` html
<input type="date" class="form-control">
```

Aby określiść skórkę dla wtyczki selektora daty, musimy dodać konkretną klasę:

``` html
<bs-date-input
  data-3d-date-picker="true"
  class="date-picker-theme-dark"
></bs-date-input>
```
W tym przypadku otrzymamy dwie wartości `class`:

W tym przypadku zdefiniowaliśmy dwie klasy:

- `form-control`, jest ustawiony przez komponent w jego szablonie
- `date-picker-theme-dark`, który jest przekazywany do komponentu przez jego rodzica

W przypadku większości atrybutów wartość przekazana komponentowi zastąpi wartość ustawioną przez komponent. Przykładowo przekazanie komponentowi `type="large"` nadpisze `type="date"`. Na szczęście atrybuty `class` i `style` są nieco mądrzejsze i obie wartości zostana połączone, tworząc ostateczną wartość: `form-control date-picker-theme-dark`.

## Zdarzenia niestandardowe

Przyzwyczaiłeś się zapewne do przekazywania props z rodzica do dziecka, a jak się skomunikować z rodzicem w razie potrzeby? To jest czas na poznanie systemu niestandardowych zdarzeń Vue.

### Korzystanie z `v-on` przy zdarzeniach niestandardowych

Każda instancja Vue implementuje [interfejs zdarzeń](../api/#Instance-Methods-Events) co oznacza, że może:

- nasłuchiwać zdarzeń korzystając z: `$on(eventName)`
- emitować zdarzenia korzystając z: `$emit(eventName)`

<p class="tip">Zauważ, że system zdarzeń Vue różnie się od [EventTarget API](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget) przeglądarki. Chociaż działają podobnie, `$on` i `$emit` __nie__ są skrótami do `addEventListener` i `dispatchEvent`. </p>

Ponadto komponent nadrzędny może nasłuchiwać zdarzeń emitowanych z komponentu potomnego, używając `v-on` bezpośrednio w szablonie, w którym używany jest komponent potomny.

<p class="tip">Nie możesz użyć `$on` do nasłuchiwania zdarzeń dziecka. Trzeba skorzystać z `v-on` bezpośrednio w szablonie jak na ponizszym przykladzie.</p>

Przykład:

``` html
<div id="counter-event-example">
  <p>{{ razem }}</p>
  <button-counter v-on:inkrementuj="inkrementujRazem"></button-counter>
  <button-counter v-on:inkrementuj="inkrementujRazem"></button-counter>
</div>
```

``` js
Vue.component('button-counter', {
  template: '<button v-on:click="inkrementujLicznik">{{ licznik }}</button>',
  data: function () {
    return {
      licznik: 0
    }
  },
  methods: {
    inkrementujLicznik: function () {
      this.licznik += 1
      this.$emit('inkrementuj')
    }
  },
})

new Vue({
  el: '#counter-event-example',
  data: {
    razem: 0
  },
  methods: {
    inkrementujRazem: function () {
      this.razem += 1
    }
  }
})
```

{% raw %}
<div id="counter-event-example" class="demo">
  <p>{{ razem }}</p>
  <button-counter v-on:inkrementuj="inkrementujRazem"></button-counter>
  <button-counter v-on:inkrementuj="inkrementujRazem"></button-counter>
</div>
<script>
Vue.component('button-counter', {
  template: '<button v-on:click="inkrementujLicznik">{{ licznik }}</button>',
  data: function () {
    return {
      licznik: 0
    }
  },
  methods: {
    inkrementujLicznik: function () {
      this.licznik += 1
      this.$emit('inkrementuj')
    }
  },
})
new Vue({
  el: '#counter-event-example',
  data: {
    razem: 0
  },
  methods: {
    inkrementujRazem: function () {
      this.razem += 1
    }
  }
})
</script>
{% endraw %}

Zwróc uwagę, że w tym przykladzie komponent potomny wciąż jest całkowicie odseparowany od tego co się dzieje na zewnątrz. Jego zadaniem jest przekazywanie informacji o swojej aktywności na wypadek, gdyby to mogło dotyczyć komponentu nadrzędnego.

### Bindowanie natywnych zdarzeń do komponentu

Może się zdarzyć, że chcesz nasłuchiwać zdarzenia natywnego w głównym elemencie komponentu. W takich przypadkach możesz użyć modyfikatora `.native` dla` v-on`. Na przykład:

``` html
<my-component v-on:click.native="zrobTo"></my-component>
```

### Modyfikator `.sync`

> 2.3.0+

W niektórych przypadkach będziemy potrzebować "dwukierunkowego bindowania" dla prop, w Vue 1.x zajmował się tym modyfikator `.sync`. Jeżeli komponent potomny zmienił prop mający modyfikator `.sync`, zmiana wartości została odzwierciedlona w rodzicu. Jest to wygodne, jednak w dłuższej perspektywie prowadzi do problemów z utrzymaniem, ponieważ przerywa jednokierunkowe założenie przepływu danych: kod, który mutuje prop dziecka, pośrednio wpływa na stan rodzica.

Właśnie z tego powodu modyfikator `.sync` wycofaliśmy od wersji 2.0. Odkryliśmy jednak, że rzeczywiście istnieją przypadki, w których może to być przydatne, szczególnie w przypadku budowania komponentów wielokrotnego użytku
Zmiany wymagała **czytelność i spójność kodu dziecka, wpływającego na stan rodzica**.

W wersji 2.3.0+ przwrócilismy modyfikator `.sync` dla props, ale tym razem jako cukier składniowy, który roższerza detektor `v-on`:

Poniższy kod:

``` html
<comp :foo.sync="bar"></comp>
```

ewoluował do:

``` html
<comp :foo="bar" @update:foo="val => bar = val"></comp>
```

Wystarczy, że komponent potomny zaktualizuje `foo`, zamiast zmieniać prop, wyemituje zdarzenie:

``` js
this.$emit('update:foo', newValue)
```

### Komponenty inputów formularza korzystające z niestandardowych zdarzeń

Niestandardowe zdarzenia moga być również użyte do tworzenia niestandardowych inputów współpracujących z `v-model`. Zapamietaj:

``` html
<input v-model="costam">
```

to cukier składniowy dla:

``` html
<input
  v-bind:value="costam"
  v-on:input="costam = $event.target.value">
```

Zapis przy wykorzystaniu komponentu upraszcza się do:

``` html
<custom-input
  :value="costam"
  @input="value => { costam = value }">
</custom-input>
```

A więc, aby komponent działał z `v-model`, powinien (konfiguracja dostępna od 2.2.0+):

- akceptować prop `value`,
- emitować zdarzenie `input` z nową wartością.

Zobaczmy to w akcji przy inpucie do wprowadzenia ceny:

``` html
<currency-input v-model="price"></currency-input>
```

``` js
Vue.component('currency-input', {
  template: '\
    <span>\
      $\
      <input\
        ref="input"\
        v-bind:value="value"\
        v-on:input="updateValue($event.target.value)">\
    </span>\
  ',
  props: ['value'],
  methods: {
    // Zamiast aktualizować wartość bezpośrednio, 
    // ta metoda służy do formatowania i umieszczania
    // reguł na wartości inputów
    updateValue: function (value) {
      var formattedValue = value
        // Przycięcie pustych znaków po obu stronach
        .trim()
        // Skrócenie do dwóch miejsc po przecinku
        .slice(
          0,
          value.indexOf('.') === -1
            ? value.length
            : value.indexOf('.') + 3
        )
      // Jeśli wartość nie była już znormalizowana,
      // ręcznie ją nadpisz w celu dopasowania
      if (formattedValue !== value) {
        this.$refs.input.value = formattedValue
      }
      // Emituj wartość liczbową za pomocą zdarzenia inputu
      this.$emit('input', Number(formattedValue))
    }
  }
})
```

{% raw %}
<div id="currency-input-example" class="demo">
  <currency-input v-model="price"></currency-input>
</div>
<script>
Vue.component('currency-input', {
  template: '\
    <span>\
      $\
      <input\
        ref="input"\
        v-bind:value="value"\
        v-on:input="updateValue($event.target.value)"\
      >\
    </span>\
  ',
  props: ['value'],
  methods: {
    updateValue: function (value) {
      var formattedValue = value
        .trim()
        .slice(
          0,
          value.indexOf('.') === -1
            ? value.length
            : value.indexOf('.') + 3
        )
      if (formattedValue !== value) {
        this.$refs.input.value = formattedValue
      }
      this.$emit('input', Number(formattedValue))
    }
  }
})
new Vue({
  el: '#currency-input-example',
  data: { price: '' }
})
</script>
{% endraw %}


Powyższa implementacja jest jednak dość naiwna. Na przykład użytkownicy mogą czasem wpisywać wiele kropek, a nawet liter - fuj! Tak więc dla tych, którzy chcą zobaczyć nietrywialny przykład, oto bardziej solidny filtr walutowy:

<iframe width="100%" height="300" src="https://jsfiddle.net/chrisvfritz/1oqjojjx/embedded/result,html,js" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### Customizing Component `v-model`

> Nowe 2.2.0+

Domyślnie `v-model` w komponencie używa `value` jako prop i `input` jako zdarzenia, ale niektóre typy inputów, takie jak checkbox i radio buttony, mogą chcieć użyć prop `value` do innego celu. Użycie opcji `model` pozwala uniknąć konfliktu w takich przypadkach:

``` js
Vue.component('my-checkbox', {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    checked: Boolean,
    // to pozwala użyć `value` prop do innych celów
    value: String
  },
  // ...
})
```

``` html
<my-checkbox v-model="foo" value="jakaś wartość"></my-checkbox>
```

Powyższe będzie równoznaczne z:

``` html
<my-checkbox
  :checked="foo"
  @change="val => { foo = val }"
  value="jakaś wartość">
</my-checkbox>
```
<p class="tip">Zauważ, że musisz nadal wyraźnie zadeklarować prop `checked`.</p>

### Komunikacja poza relacjami rodzic-dziecko

Czasami dwa komponenty potrzebują się skomunikować, ale nie są w wzajemnej relacji rodzic-dziecko. Najprostszym rozwiązaniem jest wykorzystanie pustej instacji Vue jako centralna magistrala zdarzeń:  

``` js
var bus = new Vue()
```
``` js
// metoda w komponencie A
bus.$emit('id-selected', 1)
```
``` js
// uchwyt w komponencie B
bus.$on('id-selected', function (id) {
  // ...
})
```

W bardziej skomplikowanych przypadkach, należy rozważyć użycie dedykowanego [szablon zarządzania zdarzeniami](state-management.html).

## Dystrybucja zawartości za pomocą slotów

Pracując z komponentami, często układa się je w taki sposób:

``` html
<app>
  <app-header></app-header>
  <app-footer></app-footer>
</app>
```

Zwróc uwagę na dwa aspekty:

1. Komponent `<app>` nie wie jaką treść otrzyma. Decyduje o tym komponent korzystający z `<app>`.

2. Komponent `<app>` bardzo często ma własny szablon.

Aby kompozycja działała, potrzebujemy sposobu na przeplatanie nadrzędnej "treści" i własnego szablonu komponentu. Jest to proces o nazwie **content distribution** (lub "transclusion", jeśli znasz Angular). Vue.js implementuje interfejs API do dystrybucji treści, który jest wzorowany na aktualnym projekcie specyfikacji [Web Components] (https://github.com/w3c/webcomponents/blob/gh-pages/proposals/Slots-Proposal.md), wykorzystujący specjalny element `<slot>" służący jako punkty dystrybucyjne dla oryginalnej treści.

### Zasięg kompilacji

Zanim przejdziemy do API, najpierw wyjaśnijmy, w jakim zakresie kompiluje się zawartość. Wyobraź sobie szablon podobny do tego:

``` html
<child-component>
  {{ komunikat }}
</child-component>
```

Czy `komunikat` powinien być powiązany z danymi rodzica czy dziecka? Odpowiedź brzmi: rodzica. Prostą zasadą ilustrującą zasięg komponentu jest:

> Wszystko w szablonie rodzica jest kompilowane w zakresie rodzica; wszystko w szablonie dziecka jest kompilowane w zasięgu dziecka.

Typowym błędem jest próba powiązania dyrektywy z właściwością / metodą dziecka w szablonie rodzica:

``` html
<!-- NIE zadziała -->
<child-component v-show="someChildProperty"></child-component>
```

Zakładając, że `someChildProperty` jest właściwością komponentu potomnego, powyższy przykład nie zadziała. Szablon rodzica nie jest świadomy stanu komponentu potomnego.

Jeśli chcesz powiązać dyrektywy z zasięgu dziecka z węzłem głównym komponentu, powinieneś to zrobić w szablonie komponentu dziecka:

``` js
Vue.component('child-component', {
  // this does work, because we are in the right scope
  template: '<div v-show="someChildProperty">Child</div>',
  data: function () {
    return {
      someChildProperty: true
    }
  }
})
```

Podobnie, dystrybuowana zawartość zostanie skompilowana w zasięgu rodzica.

### Pojedyńczy slot

Treść nadrzędna zostanie **odrzucona**, chyba że szablon komponentu potomnego zawiera co najmniej jeden element `<slot>`. Gdy jest tylko jeden taki element bez atrybutów, cały fragment treści zostanie wstawiony na swoje miejsce w DOM, zastępując sam slot.

Wszystko, co pierwotnie znajdowało się w tagach `<slot>`, jest uważane za **treść zastępczą**. Treść zastępcza jest kompilowana w zasięgu dziecka i będzie wyświetlana tylko wtedy, gdy element gospodarza jest pusty i nie ma w nim treści do wstawienia.

Załóżmy, że mamy komponent `my-component` z poniższym szablonem:

``` html
<div>
  <h2>Tytuł dziecka</h2>
  <slot>
    To zostanie wyświetlone wyłącznie, gdy żadna zawartość
    nie jest dystrybuowana.
  </slot>
</div>
```

I rodzica korzystającego z komponentu:

``` html
<div>
  <h1>Tytuł rodzica</h1>
  <my-component>
    <p>Oryginalna zawartość</p>
    <p>Więcej oryginalnej zawartości</p>
  </my-component>
</div>
```

Wyrenderowany kod będzie wyglądał nastepująco:

``` html
<div>
  <h1>Tytuł rodzica</h1>
  <div>
    <h2>Tytuł dziecka</h2>
    <p>Oryginalna zawartość</p>
    <p>Więcej oryginalnej zawartości</p>
  </div>
</div>
```

### Nazwy slotów

Elementy `<slot>` mają specjalny atrybut, `name`, który można wykorzystać do dalszego dostosowywania sposobu dystrybucji treści. Możesz mieć wiele slotów o różnych nazwach. Nazwany slot będzie pasować do dowolnego elementu, który ma odpowiedni atrybut `slot` w fragmencie treści.

Nadal może istnieć jeden nienazwany slot, który jest **domyślnym slotem**, służącym jako punkt wyjścia dla wszystkich niedopasowanych treści. Jeśli nie ma domyślnego slotu, niedopasowane treści zostaną odrzucone.

Wyobraź sobie, że mamy komponent `app-layout` z następującym szablonem:

``` html
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```

Składnia rodzica:

``` html
<app-layout>
  <h1 slot="header">Tutaj może być tytuł strony</h1>

  <p>Akapit w głównej zawartości.</p>
  <p>I jeszcze jeden.</p>

  <p slot="footer">Dane kontaktowe</p>
</app-layout>
```

Wyrenderowany kod:

``` html
<div class="container">
  <header>
    <h1>Tutaj może być tytuł strony</h1>
  </header>
  <main>
    <p>Akapit w głównej zawartości.</p>
    <p>I jeszcze jeden.</p>
  </main>
  <footer>
    <p>Dane kontaktowe</p>
  </footer>
</div>
```

Interfejs API do dystrybucji treści jest bardzo przydatnym mechanizmem podczas projektowania komponentów, które mają być komponowane razem.

### Slot o ograniczonym zasięgu

> Nowość w 2.1.0+

Slot o ograniczonym zasięgu jest specjalnym rodzajem slotu, który działa jako szablon wielokrotnego użytku (do którego można przekazywać dane) zamiast już wyrenderowanych elementów.

W komponencie potomnym, przekaż dane do slotu tak, jakbyś przekazywał prop do komponentu:

``` html
<div class="dziecko">
  <slot text="dziecko wita się"></slot>
</div>
```

Jeżeli w elemencie nadrzędnym jest element `<template>` ze specjalnym atrybutem `slot-scope`, to oznacza, że jest to szablon dla slotu o ograniczonym zasięgu. Wartość `slot-scope` jest używana jako nazwa zmiennej tymczasowej, która przechowuje obiekt prop przekazany przez potomka:

``` html
<div class="rodzic">
  <child>
    <template slot-scope="props">
      <span>rodzic wita się</span>
      <span>{{ props.text }}</span>
    </template>
  </child>
</div>
```

Jeżeli wyrenderujemy powyższe, uzyskamy:

``` html
<div class="rodzic">
  <div class="dziecko">
    <span>rodzic wita się</span>
    <span>dziecko wita się</span>
  </div>
</div>
```

> W 2.5.0+, `slot-scope` nie jest już ograniczone do `<template>` i może być użyte w dowolnym elemencie lub komponencie.

Bardziej typowym prykładem zastosowania slotu o ograniczonym zasięgu, jest komponent listy, pozwalający konsumentowi na ustalenie jak każdy z elementów listy ma być renderowany:

``` html
<my-awesome-list :items="items">
  <!-- slot o ograniczonym zasięu również może mieć nazwę -->
  <li
    slot="item"
    slot-scope="props"
    class="my-fancy-item">
    {{ props.text }}
  </li>
</my-awesome-list>
```

I szablon dla komponentu listy:

``` html
<ul>
  <slot name="item"
    v-for="item in items"
    :text="item.text">
    <!-- tutaj treść zastępcza -->
  </slot>
</ul>
```

#### Destrukturyzacja

wartość `slot-scope` jest w rzeczywistości poprawnym wyrażeniem JavaScript, które może być argumentem funkcji. Oznacza to, że w obsługiwanych środowiskach (w komponentach pojedynczego pliku lub w nowoczesnych przeglądarkach) można również użyć destrukturyzacji z ES2015:

``` html
<child>
  <span slot-scope="{ tekst }">{{ tekst }}</span>
</child>
```

## Komponenty dynamiczne

Możesz wybrać punktu montowania i dynamicznie przełączać komponenty w elemencie `<component>` łącząc się z jego atrybutem `is`:

``` js
var vm = new Vue({
  el: '#example',
  data: {
    currentView: 'home'
  },
  components: {
    home: { /* ... */ },
    posts: { /* ... */ },
    archive: { /* ... */ }
  }
})
```

``` html
<component v-bind:is="currentView">
  <!-- jeżeli zmieni się vm.currentView zmieni się komponent! -->
</component>
```

Jeżeli wolisz, możesz również bindować bezpośrednio obiekt komponentu:

``` js
var Home = {
  template: '<p>Witaj w domu!</p>'
}

var vm = new Vue({
  el: '#example',
  data: {
    currentView: Home
  }
})
```

### `keep-alive`

Jeżeli chcesz zachować już nie wyświetlane komponenty możesz zachować ich stan lub zapobiec ponownemu renderowaniu, opakuj komponent w element `<keep-alive>`:

``` html
<keep-alive>
  <component :is="currentView">
    <!-- nieaktywny komponent pozostanie w cache! -->
  </component>
</keep-alive>
```

Więcej informacji o `<keep-alive>` znajdziesz w [dokumentacji API](../api/#keep-alive).

## Różne

### Wytwarzanie komponentów wielokrotnego użytku

Podczas tworzenia komponentów dobrze jest pamiętać, czy zamierzasz je później wykorzystać gdzieś później. Komponenty jednorazowe mogą być ściśle sprzężone, ale komponenty wielokrotnego użytku powinny definiować czysty interfejs publiczny i nie zakładać żadnych założeń dotyczących kontekstu, w którym są używane

Interfejs API komponentu Vue składa się z trzech części - prop, zdarzenia i sloty:

- **Prop** pozwalają środowisku zewnętrznemu przekazywać dane do komponentu

- **Zdarzenia** pozwalają komponentowi wpływać na zewnętrzne środowisko

- **Sloty** pozwalają zewnętrznemu środowisku na wypełnianie komponentu treścią.

Dzięki dedykowanym skróconym składniom dla `v-bind` i `v-on`, intencje mogą być wyraźnie i zwięźle przedstawione w szablonie:

``` html
<my-component
  :foo="baz"
  :bar="qux"
  @event-a="zrobTo"
  @event-b="zrobTamto"
>
  <img slot="icon" src="...">
  <p slot="main-text">Witaj!</p>
</my-component>
```

### Identyfikator bezpośredniego odwołania do komponentu potomnego

Pomimo istnienia prop i zdarzeń, czasami możesz potrzebować bezpośredniego dostępu do komponentu potomnego w JavaScript. Aby to osiągnąć, musisz przypisać identyfikator referencyjny do komponentu potomnego za pomocą `ref`. Na przykład:

``` html
<div id="rodzic">
  <user-profile ref="profil"></user-profile>
</div>
```

``` js
var rodzic = new Vue({ el: '#rodzic' })
// odwołanie do instancji komponentu potomnego
var dziecko = rodzic.$refs.profile
```

Kiedy `ref` jest używane razem z `v-for`, otrzymasz odpowiedź, która będzie tablicą zawierającą komponenty potomne, odzwierciedlające źródło danych.

<p class="tip">`$ refs` są emitowane tylko po wyrenderowaniu komponentu i nie są reaktywne. Jest to jedynie luka ewakuacyjna do bezpośredniej manipulacji dziećmi - powinieneś unikać używania `$ refs` w szablonach lub właściwościach wyliczonych.</p>

### Komponenty asynchroniczne

W dużych aplikacjach możemy potrzebować podzielić aplikację na mniejsze elementy i ładować komponent z serwera tylko wtedy, gdy jest rzeczywiście potrzebny. Aby to ułatwić, Vue pozwala ci zdefiniować twój komponent jako funkcję fabryczną, która asynchronicznie wywołuje Twój komponent. Vue uruchomi funkcję fabryczną, tylko gdy komponent rzeczywiście będzie wymagał renderowania i będzie buforował wynik dla przyszłych ponownych renderowań. Na przykład:

``` js
Vue.component('async-przyklad', function (resolve, reject) {
  setTimeout(function () {
    // Przekaże definicję componentu jako wywołanie zwrotne `resolve`
    resolve({
      template: '<div>Jestem asynchroniczny!</div>'
    })
  }, 1000)
})
```

Funkcja fabryczna otrzymuje wywołanie zwrotne `resolve`, które powinno zostać wywołane po pobraniu definicji komponentu z serwera. Możesz także zdefiniować `reject(reason)`, aby wskazać, że ładowanie się nie powiodło. `setTimeout` jest użyte tylko przykładowo, sposób pobrania komponentu zależy od Ciebie. Jednym z zalecanych sposobów jest użycie komponentów async razem z [funkcją podziału kodu Webpacka](https://webpack.js.org/guides/code-splitting/):

``` js
Vue.component('async-webpack-przyklad', function (resolve) {
  // Tutaj jest wymagana składnia instruująca Webpack,
  // który automatycznie podziel twój kod wyjściowy na pakiety, 
  // ładowanie za pomocą Ajax.
  require(['./moj-komponent-async'], resolve)
})
```

Możesz również zwrócić `Promise` w funkcji fabrycznej, dzięki składni Webpack 2 + ES2015:

``` js
Vue.component(
  'async-webpack-przyklad',
  // Funkcja `import` zwraca `Promise`.
  () => import('./moj-komponent-async')
)
```

Korzystając z [rejestracji lokalnej](components.html#Rejestracja-lokalna), możesz zdefiniować funkcję, która zwraca `Promise`:

``` js
new Vue({
  // ...
  components: {
    'moj-komponent': () => import('./moj-komponent-async')
  }
})
```
<p class = "tip"> Jeśli korzystasz z<strong> Browserify </strong> i chciałbyś użyć komponentów asynchronicznych, nie mam dobrych wieści: jego autor [wyraźnie powiedział](https://github.com/substack/node-browserify/issues/58#issuecomment-21978224), że ładowanie asynchroniczne nigdy nie będzie oficjalnie wspierane przez Browserify. Społeczność Browserify znalazła [kilka obejść] (https://github.com/vuejs/vuejs.org/issues/620), które mogą być pomocne dla już istniejących, rozbudowanych aplikacji. W przypadku wszystkich innych scenariuszy zalecamy używanie pakietu Webpack do wbudowanej obsługi asynchronicznej. </p>

### Zaawansowane komponenty asynchroniczne

> Nowość w 2.3.0+

Począwszy od wersji 2.3.0+ wbudowany komponent asynchroniczny może również zwracać obiekt w poniższym formacie:

``` js
const AsyncComp = () => ({
  // Komponent do pobrania. Powinien być promesą
  component: import('./MyComp.vue'),
  // Komponent do użycia w czasie, gdy komponent asynchroniczny jest wczytywany
  loading: LoadingComp,
  // Komponent do użycia w razie problemu z ładowaniem
  error: ErrorComp,
  // Opóźnienie pokazania komponentu, domyślnie: 200 ms.
  delay: 200,
  // Komponent ErrorComp będzie wyświetlony 
  // jeżeli timeout jest zdefiniowany i osiągnięty.
  // Domyślnie nieskonczoność.
  timeout: 3000
})
```

Zauważ, że gdy użyjesz go jako komponentu routującego `vue-router`, właściwości te zostaną zignorowane, ponieważ komponenty asynchroniczne są rozwiązywane z góry przed rozpoczęciem routingu. Musisz także użyć `vue-router` 2.4.0+, jeśli chcesz użyć powyższej składni dla komponentów routujących.

### Konwencja nazewnictwa komponentów

Rejestrując komponenty (lub props), możesz używać kebab-case, camelCase, lub PascalCase.

``` js
// w definicji komponentu
components: {
  // rejestracja z użyciem kebab-case
  'kebab-cased-component': { /* ... */ },
  // rejestracja z użyciem camelCase
  'camelCasedComponent': { /* ... */ },
  // rejestracja z użyciem PascalCase
  'PascalCasedComponent': { /* ... */ }
}
```
Jednak w szablonach HTML musisz używać ich odpowiedników w kebab-case:

``` html
<!-- zawsze używaj kebab-case w szablonach HTML -->
<kebab-cased-component></kebab-cased-component>
<camel-cased-component></camel-cased-component>
<pascal-cased-component></pascal-cased-component>
```

Korzystając z szablonów łańcuchowych nie obowiązują Cię te restrykcjie. To oznacza, że nawet w szablonie możesz odwoływać się do swoich komponentów z użyciem:

- kebab-case
- camelCase lub kebab-case jeżeli komponent jest zdefiniowany z użyciem camelCase
- kebab-case, camelCase lub PascalCase jeżeli komponent jest zdefiniowany z użyciem PascalCase

``` js
components: {
  'kebab-cased-component': { /* ... */ },
  camelCasedComponent: { /* ... */ },
  PascalCasedComponent: { /* ... */ }
}
```

``` html
<kebab-cased-component></kebab-cased-component>

<camel-cased-component></camel-cased-component>
<camelCasedComponent></camelCasedComponent>

<pascal-cased-component></pascal-cased-component>
<pascalCasedComponent></pascalCasedComponent>
<PascalCasedComponent></PascalCasedComponent>
```

To oznacza, że PascalCase jest najbardziej uniwersalną _konwencją deklaracji_ a kebeb-case najbardzoiej uniwersalną _konwencją wywołania_.

Jeżeli Twój komponent nie przyjmuje z użyciem elementów `slot`, możesz nawet użyć znaku zamknięcia tagu `/` po nazwie:

``` html
<my-component/>
```

Ponownie, to działa _tylko_ w szablonach łańcuchowych, samozamykające tagi użytkownika nie są prawidłowymi tagami HTML i natywny parser przeglądarki nie zrozumie ich.

### KOmponenty rekurencyjne

KOmponenty mogą rekurencyjnie wywoływać same siebie w swoim szablonie. Jednak mogą to zrobić tylko z opcją `name`:

``` js
name: 'unikalna-nazwa-mojego-komponentu'
```

Po zarejestrowaniu komponentu globalnie przy użyciu `Vue.component`, globalny ID jest automatycznie ustawiany jako opcja `name` komponentu.

``` js
Vue.component('unikalna-nazwa-mojego-komponentu', {
  // ...
})
```

Jeśli nie jesteś ostrożny, elementy rekurencyjne mogą również prowadzić do nieskończonych pętli:

``` js
name: 'stack-overflow',
template: '<div><stack-overflow></stack-overflow></div>'
```
Składnik taki jak powyższy spowoduje błąd "max stack size exceeded" (przekroczenia maksymalnego rozmiaru stosu), więc upewnij się, że wywołanie rekurencyjne jest warunkowe (to znaczy używa `v-if`, które ostatecznie będzie `false`).

### Kołowe odniesienia między komponentami

Załóżmy, że budujesz drzewo katalogów plików, jak w Finderze lub Eksploratorze plików. Możesz mieć komponent `tree-folder` z tym szablonem:

``` html
<p>
  <span>{{ folder.name }}</span>
  <tree-folder-contents :children="folder.children"/>
</p>
```

Następnie komponent `tree-folder-contents` z szablonem:

``` html
<ul>
  <li v-for="child in children">
    <tree-folder v-if="child.children" :folder="child"/>
    <span v-else>{{ child.name }}</span>
  </li>
</ul>
```

Kiedy przyjrzysz się uważnie, zobaczysz, że te komponenty będą w rzeczywistości potomnymi przodkami w drzewie renderowania - paradoksalnie! Podczas rejestrowania komponentów globalnie za pomocą `Vue.component`, ten paradoks jest rozwiązywany automatycznie. Jeśli ta informacja CIę satysfakcjonuje, możesz nie czytać dalej.

Jeśli jednak potrzebujesz/importujesz komponenty używając __module system__, np. przez Webpack lub Browserify dostaniesz błąd:

```
Failed to mount component: template or render function not defined.
```

Aby wyjaśnić, co się dzieje, odnieśmy się do naszych komponentów A i B. System modułów widzi, że potrzebuje A, ale najpierw A potrzebuje B, ale B potrzebuje A, ale A potrzebuje B itd. Itd. Utknął w pętli, nie wiedząc jak w pełni rozwiązać któryś z komponentów bez uprzedniego rozwiązania drugiego. Aby to naprawić, musimy nadać systemowi modułów punkt, w którym może on powiedzieć: "A potrzebuje B _finalnie_, ale nie ma potrzeby, aby najpierw rozwiązywać B".

W naszym przykładzie, zróbmy ten punkt komponentem `tree-folder`. Wiemy, że dziecko, które tworzy paradoks, jest komponentem `tree-folder-contents`, więc poczekamy, aż uchwyt cyklu `beforeCreate` zauważy je:

``` js
beforeCreate: function () {
  this.$options.components.TreeFolderContents = require('./tree-folder-contents.vue')
}
```

Problem rozwiązany!

### Szablny liniowe

Gdy specjalny atrybut `inline-template` jest obecny w komponencie podrzędnym, komponent użyje jego wewnętrznej zawartości jako szablonu, zamiast traktować go jako rozproszoną treść. Pozwala to na bardziej elastyczne tworzenie szablonów.

``` html
<my-component inline-template>
  <div>
    <p>To jest kompilowane jako własny szablon komponentu.</p>
    <p>Nie jako treść wymoszona przez rodzica.</p>
  </div>
</my-component>
```

Jednakże `inline-template` sprawia, że zakres szablonów jest trudniejszy do zrozumienia. Najlepszą metodą jest definiowanie szablonów wewnątrz komponentu za pomocą opcji `template` lub elementu `template` w pliku `.vue`.

### X-Templates

Innym sposobem definiowania szablonów wewnątrz elementu jest skrypt typu `text/x-template`, a następnie odniesienie do szablonu przez id. Na przykład:

``` html
<script type="text/x-template" id="hello-world-template">
  <p>Hej hej hej</p>
</script>
```

``` js
Vue.component('hello-world', {
  template: '#hello-world-template'
})
```

Mogą być użyteczne w przypadku wersji demonstracyjnych z dużymi szablonami lub w bardzo małych aplikacjach, ale w przeciwnym razie należy ich unikać, ponieważ oddzielają szablon od reszty definicji komponentu.

### Tanie komponenty statyczne z `v-once`

Renderowanie prostych elementów HTML jest bardzo szybkie w Vue, ale czasami możesz mieć komponent zawierający **dużo** statycznej treści. W takich przypadkach można się upewnić, że jest on sprawdzany tylko raz, a następnie buforowany przez dodanie do elementu głównego dyrektywy `v-once`, na przykład:

``` js
Vue.component('regulamin', {
  template: '\
    <div v-once>\
      <h1>Regulamin</h1>\
      ... mnóstwo statycznej zawartości ...\
    </div>\
  '
})
```
