---
title: Wartości wyliczone i obserwatorzy
type: guide
order: 5
---

## Wartości wyliczone

Wyrażenia w szablonie są bardzo wygodne, ale nadają się jedynie do prostych operacji. Zbyt duża ilość logiki w szablonach sprawi, że się nadmiernie rozrosną i stana się trudne do zarządzania. Przyjrzyj się poniższemu przykładowi:

``` html
<div id="example">
  {{ message.split('').reverse().join('') }}
</div>
```

Ten sablon nie jest już prosty ani dekalaratywny. Nie wystarczy rzut oka, aby zrozumieć, że powoduje wyświetlenie komunikatu  `message` w odwróconym szyku znaków. Problem narasta kiedy decydujesz się na załączenie odwróconego komunikatu więcej niż raz.

Właśnie dlatego przy złożonej logice powinno się korzystać z **wartości wyliczonych**.

### Prosty przykład

``` html
<div id="example">
  <p>Oryginalny komunikat: "{{ message }}"</p>
  <p>Wyliczony odwrócony komunikat: "{{ reversedMessage }}"</p>
</div>
```

``` js
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // wyliczony getter
    reversedMessage: function () {
      // `this` wskazuje na instancję vm
      return this.message.split('').reverse().join('')
    }
  }
})
```

Wynik:

{% raw %}
<div id="example" class="demo">
  <p>Oryginalny komunikat: "{{ message }}"</p>
  <p>Wyliczony odwrócony komunikat: "{{ reversedMessage }}"</p>
</div>
<script>
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    reversedMessage: function () {
      return this.message.split('').reverse().join('')
    }
  }
})
</script>
{% endraw %}

Zadeklarowaliśmy tu wyliczoną właściwość `reversedMessage`. Funkcja może być użyta jako gette dla właściwości `vm.reversedMessage`:

``` js
console.log(vm.reversedMessage) // => 'olleH'
vm.message = 'Goodbye'
console.log(vm.reversedMessage) // => 'eybdooG'
```

Możesz otworzyć konsolę i pobawić się tym przykładem samodzielnie. Wartość `vm.reversedMessage` jest zawsze zależna od wartości `vm.message`.
Możesz bindować dane to wyliczonych wartości w szablonach tak jak zwykłą właściwość. Vue jest świadome, że `vm.reversedMessage` jest zależne od `vm.message` więc zaktualizuje wszystki połączenia z `vm.reversedMessage` jeżeli `vm.message` się zmieni. Najlepsze w tym jest to, że własnie stworzyliśmy relację zależności deklaratywnie: wyliczona funkcja getter działa w sposób przewidywalny więc jest łatwiejsza do testowania i zrozumienia.

### Wyliczone cachowanie kontra metody

Jak pamiętasz osiągneliśmy taki sam rezultat definiując metodę i wywołując ją w wyrażeniu:

``` html
<p>Odwrócony komunikat: "{{ reverseMessage() }}"</p>
```

``` js
// w komponencie
methods: {
  reverseMessage: function () {
    return this.message.split('').reverse().join('')
  }
}
```

Zamiast wyliczonej własności, możesz zdefiniować taką samą funkcję jako metodę. Oba podejścia dają ten sam rezultat. Jednak jest pena różnica **własności wyliczone są cachowane bazując na ich zależnościach**. Własności wyliczone będą podlegać aktualizacji dopiero gdy ich zależności ulegną zmianie. To oznacza, że tak długo jak wiadomość `message` nie zmieni się, każde wywołanie `reversedMessage` zwróci natychmiast poprzednią wartość wyliczoną bez uruchamiania funkcji.

To również oznacza, że poniższa własność wyliczona nigdy nie zostanie zaktualizowana, ponieważ `Date.now()` nie jest reaktywną zależnością:

``` js
computed: {
  now: function () {
    return Date.now()
  }
}
```

Dla porównania, wywołanie metody **zawsze** uruchomi tę funkcję, gdy nastąpi ponowne renderowanie.

Do czego potrzebuję cachowania? Wyobraź sobie, skomplikowanie wyliczoną właściwość **A**, która wymaga wielu wyliczeń na wielu pętlach w tablicy. Mając inna właściwość zależną od **A**, bez cachowania wywołamy wyliczanie **A** dużo więcej razy niż to jest potrzebne! W przypadkach, nie wymagających cachowania, użyj metody.

### Wyliczone własności kontra obserowane własności

Vue zapewnia bardziej generyczny sposób na obserwowanie i reagowania na zmiany danych w instancji Vue: **obserwowane własności**. Jeżeli masz dane, które muszą być modyfikowane bazując na zmianach innych danych kuszące jest nadużywanie `watch` - zwłaszcza jeżeli znasz Angulara. Jednak często lepszym rozwiązaniem jest uzycie wyliczonych własności zamiast wywołania zwrotnego `watch`. Spójrz na poniższy przykład:

``` html
<div id="demo">{{ imieNazwisko }}</div>
```

``` js
var vm = new Vue({
  el: '#demo',
  data: {
    imie: 'Foo',
    nazwisko: 'Bar',
    imieNazwisko: 'Foo Bar'
  },
  watch: {
    firstName: function (val) {
      this.imieNazwisko = val + ' ' + this.nazwisko
    },
    lastName: function (val) {
      this.imieNazwisko = this.imie + ' ' + val
    }
  }
})
```

Powyży kod jest bardzo rozbudowany i mało elegancki w porównaniu do wersji wykorzystującej wyliczone właściwości:

``` js
var vm = new Vue({
  el: '#demo',
  data: {
    imie: 'Foo',
    nazwisko: 'Bar'
  },
  computed: {
    imieNazwisko: function () {
      return this.imie + ' ' + this.nazwisko
    }
  }
})
```

Duzo lepiej, nieprawdaż?

### Wyliczony setter

właściwości wyliczone domyslnie są uzywane jako getter, ale mozna ich używać jako setter, w razie potrzeby:

``` js
// ...
computed: {
  imieNazwisko: {
    // getter
    get: function () {
      return this.imie + ' ' + this.nazwisko
    },
    // setter
    set: function (nowaWartosc) {
      var osoba = nowaWartosc.split(' ')
      this.imie = osoba[0]
      this.nazwisko = osoba[osoba.length - 1]
    }
  }
}
// ...
```

Jezeli wydasz polecenie `vm.imieNazwisko = 'Jan Nowak'`, zostanie wywołany setter, który zaktualizuje `vm.imie` i `vm.nazwisko`.

## Obserwatorzy

W większości przypadków własności wyliczone są dobrym rozwiązaniem, jednak zdarzają się sytuacje wymagające użycia obserwatora. Własnie dlatego Vue zapewnia bardziej generyczną technikę reagowania na zmiany danych: opcję `watch`. Jesto najbardziej uzyteczne przy asynchronicznych lub wymagających operacjach wywoływanych zmianą danych.

np:

``` html
<div id="watch-example">
  <p>
    Zadaj pytanie zamknięte:
    <input v-model="pytanie">
  </p>
  <p>{{ odpowiedz }}</p>
</div>
```

``` html
<!-- Dzisiaj jest do dyspozycji rozbudowany ekosystem bibliotek ajax -->
<!-- i kolekcji użytecznych metod ogólnego zastosowania, rdzeń Vue -->
<!-- jest w stanie wykorzystwać je, bez wynajdowania na nowo. Daje to  -->
<!-- również deweloperom swobodę korzystania z narzędzi, które znają. -->
<script src="https://cdn.jsdelivr.net/npm/axios@0.12.0/dist/axios.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/lodash@4.13.1/lodash.min.js"></script>
<script>
var przykladObserwatora = new Vue({
  el: '#watch-example',
  data: {
    pytanie: '',
    odpowiedz: 'Nie jestem w stanie odpowiedzieć, jeżeli nie zadasz pytania!'
  },
  watch: {
    // przy każdej zmianie pytania ta funkcja zostanie wywołana
    pytanie: function (nowePytanie, starePytanie) {
      this.odpowiedz = 'Czekam aż skończysz pisać...'
      this.pobierzOdpowiedz()
    }
  },
  methods: {
    // _.debounce jest funkcją z biblioteki loadash, za jej pomocą
    // można określić ile razy dana operacja może być wykonana.
    // W tym przypadku chcemy ograniczyć liczbę odwołań do yesno.wtf/api,
    // w oczekiwaniu na zakończenie wpisywania dancyh przez użytkownika
    // przed wygenerowaniem żądania ajax. Aby się dowiedzieć więcej
    // o metodzie _.debounce (i jej kuzynie _.throttle)
    // odwiedź: https://lodash.com/docs#debounce

    pobierzOdpowiedz: _.debounce(
      function () {
        if (this.pytanie.indexOf('?') === -1) {
          this.odpowiedz = 'Pytania zazwyczaj kończą się pytajnikiem. ;-)'
          return
        }
        this.odpowiedz = 'Myślę...'
        var vm = this
        axios.get('https://yesno.wtf/api')
          .then(function (response) {
            vm.odpowiedz = _.capitalize(response.data.answer)
          })
          .catch(function (error) {
            vm.odpowiedz = 'Błąd! API niesotępne. ' + error
          })
      },
      // To jest czas w którym czekamy na zakończenie wpisywania
      // przez użytkownika, w milisekundach.
      500
    )
  }
})
</script>
```

Result:

{% raw %}
<div id="watch-example" class="demo">
  <p>
    Zadaj pytanie zamknięte:
    <input v-model="pytanie">
  </p>
  <p>{{ odpowiedz }}</p>
</div>
<script src="https://cdn.jsdelivr.net/npm/axios@0.12.0/dist/axios.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/lodash@4.13.1/lodash.min.js"></script>
<script>
var przykladObserwatora = new Vue({
  el: '#watch-example',
  data: {
    pytanie: '',
    odpowiedz: 'Nie jestem w stanie odpowiedzieć, jeżeli nie zadasz pytania!'
  },
  watch: {
    // przy każdej zmianie pytania ta funkcja zostanie wywołana
    pytanie: function (nowePytanie, starePytanie) {
      this.odpowiedz = 'Czekam aż skończysz pisać...'
      this.pobierzOdpowiedz()
    }
  },
  methods: {
    // _.debounce jest funkcją z biblioteki loadash, za jej pomocą
    // można określić ile razy dana operacja może być wykonana.
    // W tym przypadku chcemy ograniczyć liczbę odwołań do yesno.wtf/api,
    // w oczekiwaniu na zakończenie wpisywania dancyh przez użytkownika
    // przed wygenerowaniem żądania ajax. Aby się dowiedzieć więcej
    // o metodzie _.debounce (i jej kuzynie _.throttle)
    // odwiedź: https://lodash.com/docs#debounce

    pobierzOdpowiedz: _.debounce(
      function () {
        if (this.pytanie.indexOf('?') === -1) {
          this.odpowiedz = 'Pytania zazwyczaj kończą się pytajnikiem. ;-)'
          return
        }
        this.odpowiedz = 'Myślę...'
        var vm = this
        axios.get('https://yesno.wtf/api')
          .then(function (response) {
            vm.odpowiedz = _.capitalize(response.data.answer)
          })
          .catch(function (error) {
            vm.odpowiedz = 'Błąd! API niesotępne. ' + error
          })
      },
      // To jest czas w którym czekamy na zakończenie wpisywania
      // przez użytkownika, w milisekundach.
      500
    )
  }
})
</script>
{% endraw %}

W powyższym przykładzie, wykorzystanie opcji `watch` wyzwala asynchroniczną operację (dostęp do API), ograniczając częstotliwość uruchomienia jej i ustawia stany pośrednie przed podaniem końcowej odpowiedzi. Zauważ, że jest to możliwe dzięki własnościom wyliczonym.

Jako dodatek do opcji `watch` możesz wykorzystać [vm.$watch API](../api/#vm-watch).
