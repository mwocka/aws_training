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

### VPC a adresy IP

Każdy VPC posiada przedział prywatnych adresów IP, z notacją CIDR czyli adresowanie za pomocą maski. Np. 0.0.0.0/0 to wszystkie adresy, 192.168.1.0/24 to 254 adresy do użycia, gdzie 5 adresów rezerwuje AWS, 4 pierwsze i ostatni. Czyli możemy użyć 249 adresów.

Każde VPC dzielimy na kilka podsieci, (skrajny przypadek - jeden VPC jedna podsieć). Każdy subnet jest stworzony za pomocą CIDR. Każda podsieć jest przechowywana w konkretnej Avability Zone.

Możemy też tworzyć własne tablice routingu. Możemy zatem wydzielić routing żeby połączyć nasz subnet do Internetu (public subnets). Jeśli nie mamy routingu do Internet gatewy to nasz subnet jest prywatny.

_Publi subnets_ - tworzymy _Internet gateway_ i podpinay do subnetu.

**NAT gateway** - czyli chcemy się dostać np. do internetu ale tylko z jednej strony.  NAT gateway musi być w podsieci publicznej, a z prywatnej podsieci ustawiamy routing na NAT gateway np. `0.0.0.0/0 <nat-id>`. 

Do prywtnych subnetów możemy podpiąć się poprzez VPN - w AWS nazywa się to _Virtual Private Gateway_.

Rekomendacją AWS jest umieszczanie wszystkiego w prywatnych subnetach i wykorzystywanie narzędzi AWS do kierowania dostępami do nich.

### Elastic Network Interfaces

Jest to wirtualny intersejs sieciowy, który może być przepinany pomiędzy maszynami EC2 w tej samej Availability Zone. Każdy intefrejs ma swój prywatny IP adres i adres MAC.

Stosowany do np. zarządzania maszyną za pomocą SSH.

Elastic Network Interfaces posiadają również _Elastic IP Adresses_ czyli mechanizm do stworzenia niezmiennego adresu IP, który jest publiczny. 5 takich adresów może być stworzonych w jednym regionie.

## Security Groups

Jest to firewall, gdzie gdy pozwalamy na wyjście to taki pakiet może też wejść tzw. statefull. Po dodaniu security group cały ruch jest zablokowany. 

Security group use case: do strony WWW pozwalam na dostęp wszystkim (0.0.0.0/0) poprzez port 443 (https). Do aplikacji _App_ dostanie się tylko Web po porcie 80. Natomiast do bazy danych dostęp jest na porcie 3306 a source jest ustawiony na te aplikacje. Pomaga to uniknąć hardcodowania IP. 

Kolejnym takim rozwiązaniem są ACL. Przypinany bezpośrednio do podsieci. Podajemy wejścia i wyjścia, i mamy możliwości _Allow_ i _Deny_. Tylko 20 reguł na jedną ACL!

Kanapka security:

Internet gateway -> route table -> ACL -> Security Groups -> Application

**WAŻNE**

1. Czy VPC ma podpięty internet gateway
2. Czy maszyna EC2 jest w podsieci, która ma routing na internet gateway
3. Czy aplikacja ma publiczny adres czy elastic IP
4. Sprawdzamy ACL i SGs

VGW (Virtual Private Gateway) taki VPN pomiędzy data center a naszym VPC. Możemy mieć też własne łącze (DX) do AWS. 1 Gb, 10 Gb lub mniejsze.

**Jak połączyć ze sobą VPC** - używamy VPC Peering, a adresy nie mogą się **nakładać**. Mam też rozwiązanie Peering Multiple VPCs, żeby połączyć więcej VPCs na raz. Nazywa się to _Transit Gateway_.

**VPC Endpoints** - prywatna rura to serwisów AWS, która może być skonfigurowana z VPC.

## Elastic Load Balancing (ELB)

Load balancer - maszyny chowamy do prywatnych sieci, a prze loadbalancer dobijamy się do naszych serwerów webowych. **ELB** używa HTTPS, HTTP, TCP i SSL. Może być wystawiony publicznie jak i nie. Rozwijane aktuanie dwa typy ELB dla warstwy 7 jak i 4. Dzięki temu mamy wysoką wydajność i różne security features. 

Wypinanie aplikacji jest stopniowe, najpierw przekierowywane są requesty a pozniej odpinana jest apka. Nie jest to domyślnie włączone. Należy włączyć **draining**.

**Load Balancer to jeden z nielicznych działających serwisów na poziomie regionów.**

## Amazon Route 53

DNS-owy serwis, który może przekierować ruch pomiędzy regionami. Bogaty zestaw możliwości, który pomaga dostarczyć naszą aplikację. Wspiera też możliwość _anycast_.

## IAM

Bezpieczeństwo konta AWS - defaultowo założone konto jest rootowe, które nie ma graniczeń. Za pomocą IAM możemy utworzyć usera z prawami administracyjnymi. Konta _roota_ nie używam! IAM pozwala na stworzenie użytkowników i grup, pozwala na zewnętrzną autoryzację itd. Stworzony user nie ma defaultowo żadnych uprawnień.

Najpierw sprawdzany jest: _Deny_ a następnie _Allow_ w nadawaniu uprawnień użytkownikowi.

**Resource-Based** - polityka bezpieczeństwa umożliwiająca dopięcie jsona z polityką bezpieczeństwa

**Identity-Based** - dodanie specjalnie uprawnień do IAM

Gdy mamy usera, który jest potrzebny na jeden dzień to możemy wykorzystać IAM roles. Role może otrzymać każdy nawet osoba, która uwierzytelnia się za pomocą Googla.

Use Cases:

* Dostęp do AWS poprzez serwisów
* Uwierzytelnienie poprzez inne serwisy
* Zmiana dostępu do serwisów dla różnych kont AWS

Na koncie administracyjnym tworzymy userów, a do kont na produkcje np. dostajemy się poprzez role.

## High Availability Factors

Mamy przygotowane i redundantne zasoby, nasza aplikacja pozwala na skalowanie oraz mamy mechanizm na odtworzenie naszej infrastruktury np. na innej zonie.

Elastyczność pozwala nam na inteligentne skalowanie infrastruktury jak również nie marnować zasobów i pieniędzy.

Dwa typy elastyczności:

* Oparte o czas - wyłączanie zasobów, które nie są wykorzystywane,
* Oparte o wolumen - używanie tyle zasobów ile jest wykorzystywane - skalowanie automatyczne

Badanie zwiększonego ruchu tyczy się monitoringu a konkretnie badanie _health checków_ czy sprawdzanie wydajności maszyn. Do monitorowania serwisów wykorzystujemy narzędzie jakim jest **CloudWatch**.

Serwisy wysyłają do CloudWatch informacje, a następnie na tej podstawie możemy uruchomić alarmy czy wysyłać powiadomienia. Możemy też wysyłać logi z maszyny do CloudWatcha gdzie następnie jest to wrzucane do jednego miejsca np. bucketa.


### Autoscaling

Usługa pozwalająca na automatyczne skalowanie różnych serwisów. Ubijanie maszyn działa w taki sposób, że ma connection draining, żeby użytkownicy nie zauważyli zmiany. Autoscaling ma zasięg regionalny. Możemy używać skalowania w oparciu o metryki, w oparciu o konkretne dni lub za pomocą machine learning.

Bazy danych się również skalują, ale za pomocą tworzenia replik. W przypadku Aurory do nawet 15 replik naszej bazy. Bazy no-sql skalują się lepiej.

## Automatyzacja

Możemy nie tylko "klikać" ale również możemy opisać infrastrukturę za pomocą kodu. Rekomendacją AWS jest tworzenie infry za pomocą kodu wtedy możemy to wersjonować, pozwolić na audyty itp.

AWS daje nam CloudFormation co pozwala na zapisanie infrastruktury za pomocą yaml-a lub jsona. Nazywa się to **Infrastructure as Code**.
 
# AWS training - dzień trzeci

## Cashowanie

Stosowane do skrócenia czasu pobierania treści. Jest to CDN - musimy się zastanowić co jest warte cashowania. Cashowanie jest potrzebne ponieważ mamy większe zadowolenie klienta. 

## Sticky sessions

Trzymanie sesji, żeby nie ubić maszyny, na której np. ktoś robi zakupy. Dlatego sesje trzymamy np. w dynamoDB niestety to wciąż są stracone jakieś milisekundy. Dlatego AWS dodał nowy komponent Amazon DynamoDB Accelerator, co pozwala na zejście do mikrosekund.

## ElastiCashe

Kolejny serwis do trzymania cashu. Dostajemy infrastrukture: klaster z nodami gdzie pod spodem jest _Redis_ i _Memcached_. 

## Serwisy do kolejkowania

### Amazon SQS
Komunikaty ladujące na kolejkach będą wisieć tak długo aż ktoś tego nie skasuje lub sam TTL juz minie (defaultowo 4 dni). Stosowane do tego żeby przetworzyć ogromne ilości zapytań. Są to systemy asynchroniczne. Są o wiele bardziej odporne na awarie. 

## Amazon SNS
Tworzymy topic gdzie następnie dodajemy subskrybentów (e-mail, sms, lambda i inne). Stosowany do alertów do powiadomień itd.

## Microservisy i architektura Serverless

### Mikroserwisy

Zamiast monolitu aplikacji budujemy mikroserwisy, które zależą od siebie. Gdy mamy mikroserwisy możemy skalować różne elementy np. sklep, a koszyk pozostaje nie skalowany. Mikroserwisy mogą być napisane w różnych jezykach. Więc mamy sklep gdzie uwierzytelnienie jest napisane w _go_, a koszyk w _java_.

### ECS

Mechanizm AWS do uruchomienia obrazów dockerowych. AWS też pozwala na przechowywanie obrazów Dockerowych. Mamy cały czas możliwość skalowania tego klastra itd.

### AWS Fargate

Wirtualny klaster do zarządzania kontenerami. Mniej możliwości niż ECS ale szybciej się to stawia. Use Case: gdy chcemy na szybko coś postawić. Możemy dodać też do tego grupę skalującą.

### Nanoserwisy 

Jest to sama funkcjonalność, bez jakiekolwiek SO. Tak zwane usługi _Serverless_. Do tego w AWS wykorzystuje się _AWS Lambda_, która wspiera coraz więcej języków takich jak Node.js, Java, Go czy Ruby.

Lambda to nic innego jak kod, wraz z bibliotekami oraz configuracją. Kompatybilna z prawie wszystkimi serwisami AWS. Ograniczenie Lambdy to maksymalne czas wykonania kodu w lambdzie to 15 minut. Nie płacimy za zasoby - gdy Lambda nie jest używana to nie płacimy.