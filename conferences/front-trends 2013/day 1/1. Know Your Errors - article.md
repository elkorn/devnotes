W dniach 24-26 kwietnia w warszawskiej Soho Factory odbyla sie trzecia już konferencja Front-Trends. Kilkaset osób zjechalo by przez 3 dni przy pieknej pogodzie posluchac co do powiedzenia mają mniej lub bardziej znani przedstawiciele frontu z calego świata w szerokim zakresie tematów. Mieliśmy razem z Wojtkiem Gawronskim i Łukaszem Jarosinskim przyjemność uczestniczyć w tym wydarzeniu. Grzechem byloby nie podzielić sie ogromem przekazanej wiedzy! "Ogrom" to bardzo dobre slowo - materialu na jeden artykul jest **stanowczo za dużo**. Planuje rozlozyć calość na 3 artykuly, w każdym z których opisze prezentacje z kolejnych dni konferencji. A zatem...

I. Know Your Errors - Diogo Antunes
===

Dzien 1. rozpocząl Diogo Antunes prezentacją "Know Your Errors". Wystąpienie przybliżalo możliwości wykrywania JSowych bledow wystepujących na serwerach produkcyjnych. Glówne pytanie, które zostalo postawione brzmialo "Po co logować bledy JS jeżeli:
- praktykujesz TDD,
- prowadzisz testy Selenium,
- prowadzisz testy w przegladarce headless,
- dysponujesz zespolem testerów,
- na zewnątrz jest masa skryptów trzecich (Twitter, FB, Google Analytics),
- używasz CDNów
- użytkownicy instalują miliony pluginów?"

Istnieją oczywiście rzeczy, nad którymi nie ma kontroli, jednak cześć tego typu zagrożen można zalagodzić przez odpowiednie postepowanie. Do tego typu czynności zaliczono wdrażanie zmiennych środowiskowych uniemożliwiających uruchamianie kodu zewnetrznego bez wprowadzania zmian w źródlach na serwerze produkcyjnym. Tym sposobem możliwe jest podjecie szybkich dzialan na wypadek wystąpienia katastrofy.

Trzeba jednek być świadomym, że coś sie dzieje. Jedyną metodą (poza telefonami od wścieklych klientów) wykrycia bledow JS na produkcji jest ich odpowiednie logowanie.
Proces ten jest kilkuetapowy: 
1. Uzyskanie odpowiednich informacji o bledzie.
2. Przeslanie informacji (wraz z ew. dodatkowymi metadanymi) na serwer odpowiedzialny za logowanie.
3. Zalogowanie informacji o bledzie przez serwer.

Ekstrakcja informacji o bledzie
---

Do uzyskiwania informacji powszechnie stosowane sa 2 podejścia:

### Window.onerror

Najbardziej podstawowym podejściem jest podpiecie sie do eventu Error obiektu okna przeglądarki:

    window.onerror = function(msg, url, lineNo) {
        /* ... */
        return true;    // wyciszenie bledu we wspierajacych przegladarkach
    };

Metoda ta dziala praktycznie wszedzie, jednak ilosc informacji, ktore mozna za jej posrednictwem przekazac jest ograniczona.

### Try/catch

    try {
        throw new Error("katastrofa!");
    } catch (e) {
    // Obsluga bledu bez blokowania wątku UI.
    setTimeout(function(){
            e.stack;                        // Chrome, Firefox
            e.message;                      // Opera
            log(e.stack || e.message || e); // podejście uniwersalne
        }, 0);
    }

Warto zaznaczyć, że powinno sie wykorzystywać dostepne w środowisku typy bledów. Samo wywolanie `throw new 'katastrofa!'` pozbawia nas informacji na temat stosu.

[Ściąga z typów bledów dostepnych w środowisku JS][ErrorTypes]

Przesylanie informacji na serwer
---

Wśród metod mniej racjonalnych, aczkolwiek niewymagających JS, można wyszczególnić [image beacon (HTTP GET)][ImageBeacon] oraz [iframe (HTTP POST)][IframePost].

Najprostszym rozwiązaniem JavaScriptowym jest oczywiście `HTTP POST` za pomocą `XHR`:

    var xhr = new XMLHttpRequest();
    xhr.open('POST','SCIEZKA_SERWERA_LOGOWANIA');
    xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
    xhr.send(informacjeOBledzie);

Diogo przedstawil również bardzo interesujące podejście z wykorzystaniem funkcjonalności [śledzenia przez Google Analytics zdarzen mających miejsce na stronie][EventTacking].
Niestety jego zastosowanie ograniczone jest do sledzenia bledów w celu uzyskania informacji o ich liczbie.

    _gaq.push([
        '_trackEvent',
        'jserror',
        url + ': ' + lineNo,
        message || ''
    ]);

Zaprezentowany zostal również prosty, zrobiony 'na kolanie' dashboard do śledzenia ilości bledow na stronie w dziedzinie czasu, którego kod jest [dostepny na slajdach][Dashboard].

Najprostsze rozwiązania, mimo iż niezaprzeczalnie dobre i skuteczne, nie zawsze są wystarczające. Dlatego też zaprezentowany zostal zestaw bardziej zaawansowanych narzedzi slużących logowaniu bledów JS na produkcji.

Dedykowane narzedzia
---

### stacktrace.js

Bardzo prosta bilblioteka, umożliwiająca na wykorzystanie stack trace'ów w sposób niezależny od jakichkolwiek frameworków czy też przeglądarki.
Prosty przyklad [ze strony stacktrace][StackTrace]:

    <script type="text/javascript" src="path/to/stacktrace.js" />
    <script type="text/javascript">
        ... your code ...
        if (errorCondition) {
             var trace = printStackTrace();
             //Output however you want!
             alert(trace.join('\n\n'));
        }
        ... more code of yours ...
    </script>


### Skrypt + usluga

Opisane nizej narzedzia sa nieco bardziej zaawansowane i opieraja sie na dwóch 
komponentach - skryptu narzedziowego, dolączanego do naszej strony oraz uslugi świadczonej na serwerze dostawców. Za pomocą skryptu odbywa sie transmisja informacji o wystąpieniu bledów na serwer.

Uslugi świadczone na serwerze dostawców, udostepniane poprzez dashboardy na stronach internetowych opierają sie w wiekszości na agregacji i analizie statystycznej zaistnialych bledów wraz z możliwością ich filtracji.


#### Qbaka

W montażu skrypt narzedziowy Qbaka jest bardzo podobny do Google Analytics.
Umieszczenie skryptu klienckiego udostepnia obiekt `qbaka`, pośredniczący w raportowaniu zaistnialych bledów do serwera dostawców.

W ramach uslugi serwerowej dostepne są różne postaci wykresów przedstawiających czestotliwość pojawiania sie bledów. Oprócz wykresów dostepne do wglądu sa również statystyki związane z przeglądarkami, identyfikatorami użytkowników oraz wycinki kodu, jako sprawcy poszczególnych bledów.

Od 1. maja Qbaka w wersji darmowej jest obarczona ograniczeniem 200 raportowan bledów dziennie. Istnieje możliwość upgrade'u konta do wersji Unlimited Beta za $19 miesiecznie. 

_Co ciekawe, rejestrowalem sie tam po 1. maja i limit w profilu nadal pokazywal 30 000 raportowan bledów dziennie._

Wiecej informacji znaleźć można na stronie projektu: [https://qbaka.com/](https://qbaka.com/).

#### Errorception

Usluga świadczona przez Errorception jest nieco bardziej rozbudowana niż ta od Qbaki. Przede wszystkim zaimplementowano tu wsparcie dla service hooków. Domyślnie zdefiniowano hooki do popularnych aplikacji do zarządzania oraz komunikacji w ramach projektu:
 - Campfire,
 - HipChat,
 - Loggly,
 - PagerDuty.

Istnieje możliwość definiowania wlasnych [WebHooków][WebhookWiki], rozszerzających funkcjonalność o potencjalne możliwości analizy bledów przez dodatkowe systemy. Wiecej informacji na temat samej konfiguracji WebHooków można znaleźć w [instrukcji][WebhookManual].

Należy zaznaczyć, iż Errorception dostepne jest za darmo tylko w wersji ograniczonej czasowo do trzydziestu dni. Za dalsze użytkowanie twórca liczy sobie dość slono w porównaniu z Qbaką: 
 - $5/mies. za 500 raportowan bledów dziennie,
 - $14/mies. za 2000 raportowan bledów dziennie,
 - $29/mies. za 7500 raportowan bledów dziennie,
 - $59/mies. za 2000 raportowan bledów dziennie.

Projekt jest stale rozwijany, autor przyjmuje wnioski o nowe funkcjonalności.

Strona projektu Errorception: [http://errorception.com/](http://errorception.com/).

#### jsErrLog

Najwiekszą zaletą jsErrorLog wobec poprzedników jest fakt, iż nic nie kosztuje. Niestety, ceną w tym wypadku jest *skrajna* prostota uslugi. Strona projektu umożliwia wpisanie dowolnego adresu i obejrzenia pelnego raportu na temat jego bledów (jeżeli wdrożono skrypt narzedziowy). Brak tu pojecia konta użytkownika. Dodatkowym ograniczeniem jest forma prezentacji bledów. Dostajemy informacje na temat:
 - czasu wystapienia,
 - relatywnej ścieżki do pliku źródlowego,
 - adresu, pod którym wystąpil bląd,
 - numeru linii,
 - wiadomości bledu,
 - środowiska (system operacyjny + przeglądarka), w którym bląd wystąpil,
 - dodatkowych informacji (nie udalo mi sie zidentyfikować, co mialyby oznaczać).

 Dane te dostpne są w formacie stronicowanej tabeli HTML, pliku XML albo RSS.

Sam projekt hostowany jest na appspocie - może slużyć jako zewnetrzny serwer do przechowywania logów. :)

[Ostatni commit w historii repozytorium na Githubie][jsErrLogHistory] jest sprzed ponad roku, zatem można spokojnie uznać jsErrLog za obecnie nierozwijany.

Strona projektu: [http://jserrlog.appspot.com/](http://jserrlog.appspot.com/).

#### Muscula

Skrypt narzedziowy Muscula posiada unikalny identyfikator pozwalający serwerowi dostawców na przypisanie przeslanych za jego pośrednictwem bledów do jednego z potencjalnie wielu logów, należących do użytkownika. 
Sama usluga umożliwia przeglądanie wykresów agregujących zarejestrowane bledy w dziedzinie czasu oraz szczególowych informacji (zestaw taki jak w jsErrLog). Nowością w stosunku do poprzedników jest możliwość przeglądania trendów wystepowania poszczególnych bledów w czasie.

W kontekście funkcjonalności, Muscula leży gdzieś na środku skali pomiedzy Qbaką a jsErrorLog. Elementem, który czyni to narzedzie moim faworytem jest fakt, iż nie wymaga oplat. Co wiecej, autorzy na [stronie glównej][MusculaHomepage] informują, iż "zostanie rozeslana informacja kiedy, jeśli w ogóle, zaczniemy pobierać opaty- nawet wtedy platności nie bedą obowiązkowe.".

Krótkie demo możliwości Muscula: [http://www.youtube.com/watch?v=IkZ3Px2WWtc](http://www.youtube.com/watch?v=IkZ3Px2WWtc).
Strona projektu: [http://www.muscula.com/][MusculaHomepage].

Dodatek - Navigation Timing API
---

W związku z faktem, iż Diogo szybko uwinąl sie z glówną cześcią prezentacji, wystarczylo mu czasu na krótkie przedstawienie interfejsu [Navigation Timing API][NavigationTimingAPIMDN].
API to pozwala na zmierzenie wydajności czasowej strony internetowej (pod warunkiem, ze nie próbujemy z niego korzystać spod Safari lub Opery). W odróżnieniu od innych mechanizmów tego typu opartych na JS, Navigation Timing API pozwala na informacje o opóźnieniach w kontekście end-to-end. Można tym sposobem otrzymać znacznie dokladniejsze i bardziej znaczące informacje na temat wydajności.

Navigation timing jest w stanie zmierzyć wszystkie elementy reprezentowane przez przedstawiony w specyfikacji W3C model przetwarzania danych w komunikacji przeglądarki z serwerem.
Uproszczona wersja modelu:
![Model przetwarzania danych w komunikacji przeglądarki z serwerem][NavigationTimingModel]
([Pelny model zaproponowany przez W3C][FullNavigationTimingModel])

Diogo wyszczególnil jedynie dostepne w API wlaściwości, przechowujące informacje o stemplach czasowych wystepowania odpowiadających im zdarzen należących do modelu przetwarzania.
Obiekt zwiazany z navigation timing jest dostepny jako `window.performance.timing`.

Brakuje miejsca, by zglebiac szczególy Navigation Timing API na lamach artykulu - odsylam do [specyfikacji][NavigationTimingAPIW3C] oraz [MDNa][NavigationTimingAPIMDN]. Warto również wspomnieć, iż analiza pod kątem navigation timing jest jednym z elementów zestawu testów przeprowadzanych przez [PageSpeed](https://developers.google.com/speed/pagespeed/).


Podsumowanie
---

Diogo nie rozwlekal sie z podsumowaniem. :)
Przedstawil trzy proste, aczkolwiek jakże trafne, wnioski:
 - Loguj **WSZYSTKO** co tylko jesteś w stanie, **WSZEDZIE**.
 - Nie oczekuj, iż użytkownicy bedą wysylać Ci raporty o bledach.
 - Bądź świadomy, bądź przygotowany.

Slajdy z prezentacji dostepne są na SpeakerDeck: [https://speakerdeck.com/dicode/know-your-errors](https://speakerdeck.com/dicode/know-your-errors).

[ErrorTypes]: https://developer.mozilla.org/en/docs/JavaScript/Reference/Global_Objects/Error
[ImageBeacon]: http://www.arlocarreon.com/blog/javascript/javascript-image-beacons-for-tracking-user-interactions/
[IframePost]: http://css-tricks.com/snippets/html/post-data-to-an-iframe/
[EventTacking]: https://developers.google.com/analytics/devguides/collection/gajs/eventTrackerGuide
[StackTrace]: http://stacktracejs.com/
[Dashboard]: https://speakerdeck.com/dicode/know-your-errors?slide=35
[WebhookWiki]: http://en.wikipedia.org/wiki/Webhook
[WebhookManual]: http://errorception.com/api/webhook
[jsErrLogHistory]: https://github.com/Offbeatmammal/jsErrLog/commits/master?page=1
[MusculaHomepage]: http://www.muscula.com/
[NavigationTimingAPIMDN]: https://developer.mozilla.org/en-US/docs/Navigation_timing
[NavigationTimingAPIW3C]: http://www.w3.org/TR/2011/CR-navigation-timing-20110315/
[NavigationTimingModel]: navtiming_to_sitespeed.png "Model przetwarzania danych w komunikacji przeglądarki z serwerem"
[FullNavigationTimingModel]: http://www.w3.org/TR/2011/CR-navigation-timing-20110315/#processing-model