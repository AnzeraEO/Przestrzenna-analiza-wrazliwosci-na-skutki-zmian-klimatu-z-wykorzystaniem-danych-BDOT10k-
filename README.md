# Przestrzenna-analiza-wrazliwosci-na-skutki-zmian-klimatu-z-wykorzystaniem-danych-BDOT10k-
Repozytorium przedtawia przykład workflow zautomatyzowanej analizy wrażliwości przestrzeni wybranego miasta na skutki zmian klimatu (powodzie i podtopienia) z wykorzystaniem model designer w oprogramowaniu QGIS. Głównym źródłem danych wykorzystanym w analizie jest Baza Danych Obiektów Topograficznych w skali 1:10 000. 

1. Warstwy BDOT10k wykorzystane w analizie:
  - Granice miasta
  - Kompleksy użytkowania terenu:
    - kompleks oświatowy (KUOS)
    - kompleks komunikacyjny (KUKO)
    - kompleks przemysłowo-gospodarczy (KUPG)
    - kompleks zabytkowo-historyczny (KUZA)
  - Pokrycie terenu:
    - teren leśny i zadrzewiony (PTLZ)
    - plac (PTPL)
    - roślinność krzewiasta (PTRK)
    - roślinność trawiasta i uprawa rolna (PTTR)
    - uprawa trwała (PTUT)
    - zabudowa (PTZB) (dodatkowo, zostały osobno wyodrębione następujące rodzaje zabudowy: mieszkaniowa jednorodzinna i wielorodzinna, handlowo-usługowa, przemysłowo-składowa, zabudowa pozostała)
    - pozostały teren niezabudowany (PTNZ)
  - Sieć komunikacyjna:
    -  tor lub zespół torów (SKTR)
    -  droga (SKDR) (wyodrębiono drogi główne)
  - Sieć uzbrojenia terenu:
    - przewód rurowy (SUPR)
  - Sieć wodna:
    - rzeka, strumień (SWRS)
  - Budynki, budowle i urządzenia:
    - inne urządzenie techniczne (BUIT)
  - Dodatkowo z warstwy Budynki wyodrębiono pojędyjne obiekty związane z ochroną zdrowia (szpitale, przychodnie), służbami ratunkowymi, kluczowymi obiektami infrastruktury transportowej oraz obiekty związane z administracją publiczną.
2. Utworzono siatkę hekasgonalną 200m (siatka powinna być odpowiednia względem powierzchni miasta). Siatka została docięta do granicy miasta oraz dodano kolumnę z powierzchnią każdego heksagonu.
3. Warstwy wsadowe przecięto względem siatki za pomocą narzędzia Intersect.
4. Obliczono powierzchnię każdej z klas pokrycia terenu oraz udział procentowy danej klasy w komórce siatki za pomocą wyrażenia: overlay_intersects( @rodzaj pokrycia terenu , $area*100, limit:=1, sort_by_intersection_size:='des')[0] / "atrybut powierzchni siatki" / 10000 (dla obiektów poligonowych) overlay_intersects( @rodzaj pokrycia terenu,  $length , limit:=1, sort_by_intersection_size:='des')[0] (dla obiektów liniowych)
5. Ostatnim etapem analizy jest wskazanie klas wrażliwości dla danej komórki siatki w zależności od rodzaju i udziału procentowego pokrycia terenu - gdzie 1 oznacza najniższą wrażliwość, a 4 najwyższą:

CASE

  WHEN "obiekty_ochrony_zdrowia" >= 1 OR "obiekty_administracji_publicznej" >= 1 OR "obiekty_infr_transportowej" >= 1 OR "zabytki" >= 1 OR 
       "obiekty_oswiaty" >= 1 OR "obiekty_sluzb_ratunkowych" >= 1 OR "urzadzenia_techniczne" >= 1 OR
       "zabudowa_uslugowa" >= 50 OR "zabudowa_wielorodzinna" >= 50 OR "zabudowa_przemyslowa" >= 50 OR 
        "zabudowa_techniczna" >= 50 OR "plac" >= 50 THEN 4

  WHEN ("zabudowa_uslugowa" >= 1 AND "zabudowa_uslugowa" < 50) OR 
       ("zabudowa_wielorodzinna" >= 1 AND "zabudowa_wielorodzinna" < 50) OR 
       ("zabudowa_przemyslowa" >= 1 AND "zabudowa_przemyslowa" < 50) OR
       ("urzadzenia_techniczne" >= 1 AND "urzadzenia_techniczne" < 50) OR
       ("zabudowa_techniczna" >= 1 AND "zabudowa_techniczna" < 50) OR 
       ("plac" >= 1 AND "plac" < 50) THEN
    CASE
      WHEN "infrastruktura_kolejowa" >= 1 OR "infrastruktura_podziemna" >= 1 OR "infrastruktura_drogowa" >= 1 THEN 4
      ELSE 3
    END

  WHEN "zabudowa_jednorodzinna" >= 50 OR "pozostala_zabudowa" >= 50 OR 
       ("uprawa_trwala" >= 1 AND "uprawa_trwala" < 50) OR 
       ("teren_lesny_zadrzewiony" >= 1 AND "teren_lesny_zadrzewiony" < 50) OR 
       ("roslinnosc_krzewiasta " >= 1 AND "roslinnosc_krzewiasta " < 50) OR 
       ("roślinnosc_trawiasta" >= 1 AND "roślinnosc_trawiasta" < 50) THEN
    CASE
      WHEN "infrastruktura_kolejowa" >= 1 OR "infrastruktura_podziemna" >= 1 OR "infrastruktura_drogowa" >= 1 THEN 3
      ELSE 2
    END

  WHEN "uprawa_trwala" >= 50 OR "teren_lesny_zadrzewiony" >= 50 OR "roslinnosc_krzewiasta" >= 50 OR "roślinnosc_trawiasta" >= 50 OR 
       "rzeki_strumienie" >= 1 OR 
       ("zabudowa_jednorodzinna" >= 1 AND "zabudowa_jednorodzinna" < 50) OR 
       ("pozostala_zabudowa" >= 1 AND "pozostala_zabudowa" < 50) THEN 1

  ELSE 1
END

      
