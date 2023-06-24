# Identyfikacja użytkowników na podstawie odcisków palców

Celem metod biometrycznych w istniejących systemach jest uwierzytelnianie
osób zabiegających o dostęp do systemu. Uwierzytelnianie może mieć formę
weryfikacji - gdzie na wejściu dostajemy dane biometryczne jak również identyfikator
(np. login) osoby, za którą użytkownik się podaje, ale w tym projekcie
zajmujemy się trudniejszym problem - identyfikacją - gdzie na wejściu
otrzymujemy wyłącznie dane biometryczne i chcemy aby model zwrócił nam informację
czy jest to osoba zarejestrowana w systemie i jeśli tak, to która. Cechą fizyczną
wybraną do tego projektu są odciski palców.

## Dane
Dane do projektu pochodzą z datasetu dostępnego pod adresem:
https://www.kaggle.com/datasets/ruizgara/socofing

Sokoto Coventry Fingerprint Dataset to biometryczna baza odcisków palców
zaprojektowana do celów akademickich. SOCOFing składa się z 6000
obrazów odcisków palców 600 osób i zawiera unikalne atrybuty, takie jak
etykiety płci, dłoni i imienia, a także syntetycznie zmienione wersje.

W tym projekcie identyfikujemy użytkowników na podstawie odcisku prawego
palca wskazującego. Przygotowanie danych do modelu polega na wybraniu 6
obrazów spośród zmodyfikowanych dla każdej osoby. Następnie wybieramy
200 pierwszych osób z datasetu, dla których te dane są kompletne. Spośród
tych osób pierwsze 100 osób wybieramy jako użytkowników zarejestrowanych
w naszym systemie a kolejne 100 osób jako osoby spoza systemu. Dla każdej
z tych 200 osób szóste zdjęcie jest odłożone do zbioru testowego, a pierwsze
pięć zdjęć dla każdej z osób z systemu służy nam do wytrenowania modelu.

## Metoda
Do opisanego zadania wykorzystujemy syjamską sieć neuronową. W każdej
epoce trenowania dla każdego ze stu indeksów określamy trzy obrazy:
anchor, positive image oraz negative image. Anchor to zdjęcie dla osoby o
danym indeksie z listy, do którego będziemy porównywać pozostałe dwa
zdjęcia. Positive image to inne zdjęcie tej samej osoby, a negative image to
zdjęcie odcisku palca losowej innej osoby. Za każdym razem zadaniem sieci
jest zwracać prawdopodobieństwo, że dwa wejściowe obrazy są z tej samej
klasy. Otrzymujemy predykcje dla par: (anchor, positive) i (anchor, negative) i
porównując z prawdziwymi etykietami: 1.0, 0.0 liczymy Binary Cross Entropy
loss i z użyciem optymalizatora Adam aktualizujemy wagi sieci.

Sieć nazywa się syjamska ponieważ w pierwszym etapie przepuszcza oba
wejścia przez tę samą sieć zwracającą embeddingi. Ta pierwsza sieć składa
się z dwóch warstw konwolucyjnych i max_pooling i trzech warstw liniowych.

Następnie obliczamy różnice między dwoma wektorami wynikowymi i
powstały wektor przepuszczamy przez pojedynczą warstwę liniową. Po
zastosowaniu funkcji softmax uzyskujemy dwie wartości prawdopodobieństwa - druga z nich to prawdopodobieństwo należenia do klasy pozytywnej, czyli
tego, że jest to ten sam użytkownik.

Podczas testowania określamy pierwsze z pięciu dostępnych w systemie
zdjęć jako anchor image i porównujemy każde z nich z obrazem wejściowym.
Otrzymujemy listę prawdopodobieństw, jeżeli największe z nich jest większe
niż określona wartość progowa zwracamy informację, że jest to
odpowiadający tej wartości użytkownik. W przeciwnym przypadku zwracamy
informację, że jest to osoba spoza systemu.

## Wyniki
Accuracy: 0.95

Wraz ze wzrostem wartości progowej model odrzuca większą liczbę
użytkowników jako osoby spoza systemu. W przypadku zbyt niskiej wartości
progowej model może wpuszczać do systemu wiele osób spoza systemu
uznając je za jedną z osób w systemie na podstawie niskiego
prawdopodobieństwa. W przypadku zbyt dużej wartości progowej, model mógłby odrzucać wszystkich poza systemem ale również bardzo wiele osób
będących w systemie.
Optymalnym progiem z punktu widzenia miary accuracy okazał się próg 0.98
prawdopodobieństwa. Przy tak określonym progu akceptacji model osiąga
accuracy na poziomie 0.95.
Accuracy mierzone jest jako odsetek osób poprawnie zakwalifikowanych
spośród 200 osób (100 osób w systemie i 100 poza systemem).
Dla threshold = 0.98:
- Accuracy: 0.95
- F1 Score: 0.9467

