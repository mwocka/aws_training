# AWS training - dzień pierwszy

## AWS zones

Aby uniknąć problemów z dostępnością wykorzystujemy wiele _zonów_, które posiadają ten sam content (replikuje się automatycznie). Przydatne w wszelkiego rodzaju katastrofach np. gdy zaleje jeden AWS zone to drugi będzie serwować nam content aplikacji. Każdy **Availability Zone** jest od siebie odseparowany.

## AWS regions

_AWS_ dodaje często nowe regiony na dzień dzisiejszy 18 regionów. Region jest złożony z przynajmniej **dwóch** zonów. AWS sam nie replikuje danych pomiędzy regionami.

## Edge Locations

Mniejsze DC około 110 (obecnie). Dedykowane dla 3-4 serwisów dostarczonych przez _AWS_. Używane też są do _Route 53_. Zapewnia to także cashowanie informacji przynajmniej w dwóch lokacjach.

## Amazon S3

_S3_ od simple storage service. Wielka **wada** tego rozwiązania to: S3 jest dla całego regionu. Jak siądzie to cały region straci dostęp do tej usługi.

* Object-level storage
* 99,99999999999% dostępności
* Wyzwalanie wydarzeń - np. po operacji na pliku możemy zareagować

---

Use Case 1:

Przechowywanie różnych mediów np. w buckecie. Każdy obiekt wrzucony do S3 otrzymuje URL. Jest to content **prywatny** ale możemy dodać odczyt dla wszystkich, czyli upublicznić to. Pliki umieszczamy w **Bucket** czyli kontenerze na pliki w konkretnym regionie. 

Nazwa **bucketa** może być dowolna, ale nazwa jest globalna więc musi być unikalna dla całego regionu. Przykładowy url to: `https://s3-ap-[region].amazon.com/[bucket-name]/[nazwa-pliku]`. **Domyślnie content w S3 jest prywatny**. Pod S3 możemy podpinać różne polityki bezpieczeństwa, które określają kto będzie miał dostęp do naszych zasobów.

Maksymalny rozmiar pliku to 5TB. Możemy wrzucić nieskończenie wiele takich plików. 

Use Case 2:

Hostowanie stron. Odwołanie się do plików z innej domeny może wyrzucić błędy, dlatego musimy dodać konfigurację CORS.

Use Case 3:

Data lake do trzymania danych właśnie w S3. Niestety nie ma możliwości wyszukiwania w S3 co może być bardzo dużą wadą. Aby pobrać plik muszę znać ten plik. 

Use Case 4:

Narzędzie do trzymania backupów. Każdy serwis AWS wystawia API, poprzez które możemy zautomatyzować naszą pracę. Duże pliki między 5GB a 5TB są przesyłane poprzez części tzw. _multipard upload_. Uploadowanie plików można wykonać na dwa sposoby, albo będziemy to wrzucać bezpośrednio do S3 jak również do pośrednich, najbliższych serwerów, które później będą to replikować.

---

**S3** nie nadaje się do wykorzystywania file systemów. Ponieważ jest to replikowane do wielu avability zone co uniemożliwia dostępów w natychmiastowym czasie do aktualizowanych plików.

**Za co płacimy:**

+ za ilość danych,
+ za transfer do innego regionu czy Internetu,
+ za PUT, COPY, POST, LIST czy GET.

**Amazon Glacier:** coraz mniej używany, serwis, służący do backupów. Jest to serwis wbudowany w S3. Płacimy mało za storage ale nie mamy danych dostępnych online. Musimy najpierw poprosić o wyciągnięcie tych danych na zewnątrz.

**Lifecycle Policies** - polityka czasu życia obiektów. Które pomagają w zachowaniu porządku jak i oszczędności. Np. po 30 dniach plik zmieni się na S3 IA (tańszy) po 60 na Glaciera a po roku plik jest usuwany.

Use case:

Chcemy trzymać dane w chmurze - po jakim czasie mają być one usunięte!

### Wybieranie regionu

Odpowiednie przetrzymywanie danych w regionach musi być zgodne z prawem w państwie w którym się znajdujemy. Drugą istotną rzeczą jest odległość. W regionie również trzeba sprawdzić czy wszystkie serwisy _AWS-owe_ są dostępne. Dlatego trzeba **w momencie** tworzenia architektury, zdefiniować w jakich regionach środowisko będzie uruchomione. Żeby wszystkie serwisy były dostępne wszędzie.

**Za co płacimy:**

+ ceny różnią się od regionu

### Jak podpiąć domenę do S3

Najprościej za pomocą _route 53_. Nazwa domeny musi się pokrywać z nazwą bucketa.

## Amazon EC2

EC2 - _elastic cloud computer_ - czyli serwis mogący przetwarzać dane. Używany od hostingu po pełne funkcje serwera. Posiada _ulotne_ resources co dostarcza wysoką elastyczność. Czyli dostosowywanie infrastruktury do naszej aplikacji. 

**Free to make mistakes** - kiedy Amazon dostarczy lepsze funkcjonalności my w krótkim i szybkim czasie możemy się przepiąć. Nic nie stoi na przeszkodzie, żeby eksperymentować i uruchamiać nasz system na różnych typach instancji. Dlatego, że płacimy za godziny.

**AMI** czyli nasze obrazy mogą być, które są _pre-build_, dostępne z _marketplace_ jak również mogą być tworzone przez nas. Taki obraz następnie może być uruchomiony _EC2_.

Przygotowane AMI możemy wykorzystać w:

+ szybko uruchamiać nasze aplikacji,
+ backupowanie naszego softu,
+ pod spodem AMI jest S3

#### User Data dla EC2

**User Data** - mechanizm pozwalający na wstrzyknięcie na starcie konfiguracje serwera. To się uruchamia **tylko raz** przy pierwszym uruchomieniu maszyny. Wykorzystywane często do uruchamiania agenta np. puppeta, gdzie następnie puppet będzie zarządzał tą maszyną.

#### Storage dla EC2

**Instance storage** to dyski dla maszyn EC2, które są efemeryczne. Nowy system **EBS** czyli podsystem serwisu EC2, który żyje osobno i dostarcza nam dyski sieciowe. Są to dwa systemy dyskowe używane dla maszyn EC2. Instance storage jest w cenie maszyny **EC2**, a przepustowość jest dużo większa niż w przypadku **EBS**. 

Wolumeny **EBS** mogą być w różnych konfiguracjach w SSD jak i HDD. Są to różne typy wolumenów, które muszą zostać odpowiednio dobrane dla różnych usług.

**Oznaczenia Instancji EC2:**

+ m - nazwa rodziny
+ 5 - generacja
+ large - rozmiar

Typów instancji jest bardzo dużo, dlatego możemy eksperymentować nie narażając się na zakup serwerów i podzespołów.

Za co płacimy:

+ za godzine działania instancji,
    * płacimy za zużycie - dla Ubuntu i Amazon Linux naliczanie sekundowe, dla reszty naliczane godzinowe,
+ płatność lub deklaracja płatności z góry na rok lub 3 lata,
+ uruchamianie maszyn na wolnych zasobach Amazona

Największy priorytet ma Reserved Instances - czyli to co zostało z góry opłacone. W przypdaku kryzysowej sytuacji te instancje są priorytetowo uruchamiane.

Bardzo ważną rzeczą w AWS są **tagi**, którą podajemy jako klucz-wartość. Pomaga to zarządzać serwerami stworzonymi w AWS. Po tagach też możemy rozbić rachunek, co pokazuje np. ile kosztuje nas środowisko produkcyjne, a ile testowe.

## Bazy danych

Wiele czynników należy wziąć pod uwagę np.: czy baza się skaluje, ile będzie odczytów i zapisów, wymagania dotyczące storage-ów, dostępność bazy danych itp.

Typy baz danych: **relacyjne**, **nie-relacyjne**. Bazy nie relacyjne lepiej się skalują i lepiej znoszą pracę pod względem odczytu i zapisu.

### Amazon RDS

Relacjna baza danych, która wspiera skalowanie. Pod spodem może być wiele silników baz danych takie jak: MSQL czy Amazon Aurora. Ten ostatni silnik jest szybszy, replikuje dane na różnych _Avability zone_ itd.

### DynamoDB

Nie relacyjna baza danych dostarczona przez AWS. Jest szybka, łatwo skalowalna, pozwala na _serverless computing_. Możemy przypisać ile będziemy mieć odczytu i zapisu w danych tabelach.

Bezpieczeństwo w połączeniach z bazami danych zapewnia protokół SSL. Do migracji danych używamy dedykowane do tego serwisy np. AWS Database Migration Service.

# AWS Training - dzień drugi

## Networking in AWS 

VPC - moja prywatna sieć w AWS, niedostępna dla innych kont/użytkowników w AWS. Taki kontener na maszyny EC2 itp. Jest to logiczna dedykowana sieć gdzie definiujemy wszelkie reguły co może wejść i wyjść z naszej sieci.

**VPC** może być stworzone po 5 w każdym regionie. Może zawierać kilka **Availability zone** gdzie przynajmniej powinna zawierać 2 takie zony.

VPC używa się przedewszystkim do odeparowania środowiska. Np. środowisko produkcyjne i testowe jest oddzielne.

Więcej przypadków wykorzytania VPC jest na slajdach. Np. jeśli chcesz mieć 3 środowiska to załóż trzy konta na AWS i na każdym uruchom VPC.