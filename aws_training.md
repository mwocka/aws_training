# AWS training

## AWS zones

Aby uniknąc problemów z dostępnością wykorzystujemy wiele _zonów_, które posiadają ten sam content (replikuje się automatycznie). Przydatne w wszeliego rodzaju katastrofach np. gdy zaleje jeden AWS zone to drugi będzie serwować nam content aplikacji.

## AWS regions

_AWS_ dodaje często nowe regiony na dzień dzisiejszy 18 regionów. Region jest złożony z przynajmniej **dwóch** zonów. AWS sam nie replikuje danych pomiędzy regionami.

## Edge Locations

Mniejsze DC około 110 (obecnie). Detykowane dla 3-4 serwisów dostarczonych przez _AWS_. Używane też są do _Route 53_. Zapewnia to także cashowanie informacji przynajmniej w dwóch lokacjach.

## Usługi:

### Amazon S3

_S3_ od simple storage service. Wielka **wada** tego rozwiązania to: S3 jest dla całego regionu. Jak siądzie to cały region straci dostęp do tej usługi.

* Object-level storage
* 99,99999999999% dostępności
* Wyzwalanie wydarzeń - np. po operacji na pliku możemy zareagować

---

Use Case 1:

Przechowywanie różnych mediów np. w buckecie. Każdy obiekt wrzucony do S3 otrzymuje URL. Jest to content **prywatny** ale możemy dodać odczyt dla wszystkich czyli upublicznić to. Pliki umieszczamy w **Bucket** czyli konenerze na pliki w konkretnym regionie. 

Nazwa **bucketa** może być dowolna ale nazwa jest globalna więc musi być unikalna dla całego regionu. Przykładowy url to: `https://s3-ap-[region].amazon.com/[bucket-name]/[nazwa-pliku]`. **Domyślnie content w S3 jest prywatny**. Pod S3 możemy podpinać różne polityki bezpieczeństwa, które określają kto będzie miał dostęp do naszych zasobów.

Maksymalny rozmiar pliku to 5TB. Możemy wrzucić nieskończenie wiele takich plików. 

Use Case 2:

Hostowanie stron. Odwołanie się do plików z innej domeny może wyrzucić błędy dlatego musimy dodać konfigurację CORS.

Use Case 3:

Data lake do trzymania danych właśnie w S3. Niestety nie ma możliwości wyszukiwania w S3 co może być bardzo dużą wadą. Aby pobrać plik muszę znać ten plik. 

Use Case 4:

Narzędzie do trzymania backupów. Każdy serwis AWS wystawia API, poprzez które możemy zautomatyzować naszą pracę. Duże pliki między 5GB a 5TB są przesyłane poprzez części tzw. _multipard upload_. Uploadowanie plików można wykonać na dwa sposoby, albo będziemy to wrzucać bezpośrednio do S3 jak również do pośrednich, najbliższych serwerów, które poźniej będą to replikować.

---

**S3** nie nadaje się do wykorzystywania file systemów. Ponieważ jest to replikowane do wielu avability zone co uniemożliwa dostępów w natychmiastowym czasie do aktualizowanych plików.

**Za co płacimy:**

+ za ilość danych,
+ za transfer do innego regionu czy internetu,
+ za PUT, COPY, POST, LIST czy GET.

**Amazon Glacier:** coraz mniej używany, serwis, służący do backupów. Jest to serwis wbudowany w S3. Płacimy mało za storage ale nie mamy danych dostępnych online. Musimy najpierw poprosić o wyciągnięcie tych danych na zewnątrz.

