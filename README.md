# DetecTeam HAE: Hybrydowa analiza AI do wykrywania HAE

Projekt akademicki z zakresu **Data Science** i **Inżynierii AI**, demonstrujący zaawansowane techniki przetwarzania języka naturalnego (NLP) do identyfikacji cech rzadkiej choroby – **Dziedzicznego Obrzęku Naczynioruchowego (HAE)** – z niestrukturyzowanych danych medycznych.

Projekt implementuje i porównuje dwie odrębne metodologie AI:

1. **Analizę Zero-Shot (In-Context Learning):** Wykorzystanie dużych modeli językowych (LLM) jako ekspertów do analizy całego zbioru danych.

2. **Potok ETL oparty na LLM + Klasyczny ML:** Użycie LLM do transformacji (ETL) danych tekstowych na format ustrukturyzowany, a następnie trenowanie klasycznego modelu (Random Forest) w celu znalezienia najważniejszych predyktorów choroby.

## Stos Technologiczny

* **Język Programowania:** **Python**

* **Analiza Danych:** **Pandas**, **Scikit-Learn**

* **Dostęp do Modeli AI:** **OpenRouter API**

* **Modele Językowe (LLM):** **Google Gemini** (`google/gemini-2.0-flash-exp:free`) oraz **DeepSeek** (`deepseek/deepseek-chat-v3-0324:free`)

* **Przetwarzanie Danych:** **Regex (`re`)**

* **Środowisko:** **Jupyter Notebook**

## Metodologia Projektu

Projekt został podzielony na dwa równoległe podejścia analityczne w celu dogłębnego zbadania problemu.

### Podejście 1: LLM jako Ekspert (In-Context Learning)

W notebooku `full_analysis.ipynb` zaimplementowano metodę polegającą na traktowaniu LLM (Gemini/DeepSeek) jako eksperta medycznego.

1. Wczytano dwa zbiory danych (`descriptions_sick.xlsx` i `descriptions_control_group.xlsx`).

2. Całe zbiory danych zostały przekonwertowane do formatu JSON i wstrzyknięte bezpośrednio do promptu.

3. Model LLM otrzymał zadanie przeanalizowania *obu* grup pacjentów, identyfikacji unikalnych cech HAE i oceny ich istotności w jednym przebiegu (zero-shot).

4. Wygenerowane analizy tekstowe zostały zapisane (`results/full_m1.txt`, `results/full_m2.txt`).

### Podejście 2: LLM-Powered ETL + Klasyfikacja

Ta metoda wykorzystuje LLM jako narzędzie do transformacji danych (ETL) w celu przygotowania zbioru do tradycyjnego modelowania.

1. **Definicja Schematu:** W pliku `data/symptoms_description.txt` zdefiniowano ustrukturyzowany schemat 14 kluczowych symptomów HAE (np. `OS` - obrzęki skóry, `BB` - bóle brzucha, `LK` - nieskuteczność leków alergicznych).

2. **Transformacja LLM (ETL):** W notebooku `table_analysis.ipynb` stworzono pętlę, która przetwarza *każdy* opis pacjenta (z obu grup) indywidualnie.

3. **Ekstrakcja Cech:** Dla każdego pacjenta wysyłano do LLM (DeepSeek) precyzyjny prompt, prosząc o zwrócenie *wyłącznie* binarnej odpowiedzi (0 lub 1) dla każdego z 14 symptomów.

4. **Parsowanie Regex:** Odpowiedź tekstowa modelu była parsowana przy użyciu wyrażeń regularnych (`re.findall`) w celu wyekstraktowania czystego wektora binarnego (np. `[1, 1, 0, 0, ...]`).

5. **Budowa Zbioru Danych:** Wektory binarne były dodawane jako nowe wiersze do `DataFrame`, tworząc finalny, ustrukturyzowany plik `data/symptoms.csv` z dodaną kolumną celu (`HAE` = 1 lub 0).

### Analiza Wyników (z Prezentacji)

1. **Tradycyjna Analiza:** Stworzony plik `symptoms.csv` posłużył do tradycyjnej analizy (EDA) zawartej w pliku `table_explor.ipynb` – wygenerowano mapę korelacji (heatmap).

2. **Klasyczny Model ML:** Na danych tych wytrenowano model **Random Forest (Lasy Losowe)**, który posłużył do wygenerowania wykresu **ważności predyktorów (Feature Importance)**.

3. **Porównanie:** Ostateczne wyniki z obu metod (LLM jako ekspert vs. ważność cech z Random Forest) zostały ze sobą porównane.

## Jak Uruchomić Projekt

1. Sklonuj repozytorium:

    `git clone https://github.com/KarolKleba/DetecTeam_HAE.git`

    `cd DetecTeam_HAE`

3. Utwórz i aktywuj wirtualne środowisko:

      `python -m venv venv source venv/bin/activate # (Linux/Mac)`
   
      `.\venv\Scripts\activate (Windows)`

4. zainstaluj wymagane biblioteki:

      `pip install pandas requests jupyterlab`

5. Utwórz plik `config.ini` w głównym folderze projektu i dodaj swój klucz API:

      `[API] OPENROUTER_API_KEY = Twoj_Klucz_API_OpenRouter`

      `[Paths] PATH_SICK = data/descriptions_sick.xlsx PATH_CONTROL = data/descriptions_control_group.xlsx`

      `[Models] MODEL_1 = google/gemini-2.0-flash-exp:free MODEL_2 = deepseek/deepseek-chat-v3-0324:free`

6. Uruchom Jupyter Notebook, aby przeanalizować notebooki
