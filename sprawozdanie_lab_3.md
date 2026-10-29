# Szablon sprawozdania laboratoryjnego

**Przedmiot:** Uczenie głębokie w systemach rozpoznawania i przetwarzania mowy

**Zajęcia nr / temat:** Laboratorium 3 — Prognozowanie szeregów czasowych

**Autor / grupa:** Damian Skorupa, GLO3

**Data:** 29.10.2026

## 1. Cel ćwiczenia

Celem ćwiczenia była ewaluacja modelu GRU w prognozowaniu jedno- i wielokrokowym szeregu czasowego (dane sprzedażowe) oraz zbadanie wpływu długości okna wejściowego na jakość predykcji względem prostych metod bazowych.

## 2. Metoda i implementacja

Wykorzystano biblioteki PyTorch, NumPy i Matplotlib do analizy syntetycznego szeregu czasowego (1000 dni) charakteryzującego się trendem, sezonowością i szumem. Modelem predykcyjnym była jednowarstwowa sieć GRU (32 jednostki ukryte). Trening przeprowadzono dla okna 28 dni z parametrami: lr=1e-2, batch=64, max 40 epok. Zastosowano chronologiczny podział danych (70/15/15), standaryzację wyliczaną wyłącznie ze zbioru treningowego w celu uniknięcia przecieku danych oraz wczesne zatrzymywanie (early stopping). Zbadano również wpływ długości okna (3, 7 i 28 dni) na dwóch ziarnach losowości.

## 3. Wyniki

Model bazowy (3393 parametry) osiągnął minimum błędu walidacyjnego (MSE 0.0822) w 36. epoce, po czym strata zaczęła wykazywać wahania. Wczesne zatrzymanie zapobiegło przeuczeniu, co obrazuje plik "<img width="726" height="316" alt="wykres_krzywe" src="https://github.com/user-attachments/assets/2ba06071-7fce-4522-a998-45ffaa4e70da" />
". W prognozie jednokrokowej model GRU (MAE 2.91) pokonał heurystykę sezonową (3.21) i naiwną (5.44), zbliżając się do nieusuwalnej granicy szumu (2.39). W prognozie 14-krokowej sieć GRU (MAE 3.88) ustąpiła jednak metodzie sezonowej (MAE 3.52), co wizualizuje plik "<img width="687" height="316" alt="wykres_horyzont" src="https://github.com/user-attachments/assets/21f4ed4c-8979-442d-bb86-5ea3ffe40062" />
". 

| Metoda | MAE 1-krok | MAE 14-krok | RMSE 14-krok | 
| ----- | ----- | ----- | ----- | 
| **GRU** | 2.91 | 3.88 | 4.75 | 
| **Sezonowy (t-7)** | 3.21 | 3.52 | 4.42 | 
| **Naiwny (t-1)** | 5.44 | 10.09 | 11.85 | 

W eksperymencie z długością okna, wariant 3-dniowy dał bardzo słabe wyniki (MAE na teście > 5). Okno 7-dniowe przyniosło znaczną poprawę, a okno 28-dniowe (najlepsze według straty walidacyjnej) osiągnęło na teście rezultaty zbliżone do okna 7-dniowego, w granicach rozrzutu losowego.

| Okno | Ziarno | Val MSE | MAE 1-krok | MAE 14-krok | 
| ----- | ----- | ----- | ----- | ----- | 
| 3 | 0 | 0.2567 | 5.00 | 7.18 | 
| 3 | 1 | 0.2486 | 5.04 | 7.41 | 
| 7 | 0 | 0.0855 | 2.90 | 4.21 | 
| 7 | 1 | 0.0896 | 2.88 | 3.89 | 
| 28 | 0 | 0.0822 | 2.91 | 3.88 | 
| 28 | 1 | 0.0850 | 3.01 | 4.29 | 

## 4. Analiza i interpretacja

Model GRU skutecznie przewyższa reguły bazowe w predykcji jednokrokowej, ale przegrywa z regułą sezonową w prognozie wielokrokowej. Porażka ta wynika z kumulacji błędu – w podejściu autoregresyjnym nawet najmniejsze pomyłki z poprzednich kroków degradują jakość kolejnych przewidywań. Baseline sezonowy omija ten problem, sztywno kopiując wzorzec. Eksperyment uświadamia też rolę odpowiedniego kontekstu: okno 3-dniowe jest zbyt krótkie, by objąć 7-dniową sezonowość fali, natomiast skrajne poszerzenie go do 28 dni nie poprawia już istotnie wyników sieci.

## 5. Wnioski

Zaobserwowane problemy mają bezpośrednie przełożenie na modele oparte o sygnał mowy. Zbyt krótkie okno analityczne w systemach ASR uniemożliwia sieci odczytanie dłuższych zależności, jak koartykulacja. Z kolei problem katastrofalnej kumulacji błędu podczas wielokrokowej rekurencji jest tożsamy z degradacją jakości obserwowaną w autoregresyjnych modelach syntezy mowy (TTS). Sieci GRU świetnie wychwytują bliskie wzorce, jednak stabilna generacja długich sekwencji wymaga odpowiedniego kontekstu oraz mechanizmów niwelujących narastające zniekształcenia.

