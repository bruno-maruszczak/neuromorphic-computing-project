# neuromorphic-computing-project

Projekt polega na przeglądzie metod trenowania sieci trzeciej generacji oraz zaimplementowania takiej sieci realizującej zadanie klasyfikacji. 

## Metody trenowania Pulsujących Sieci Neuronowych

Metody opisane w oparciu o:

[Toward Large-scale Spiking Neural Networks: A Comprehensive Survey and Future Directions](https://doi.org/10.48550/arXiv.2409.02111)

[A critical survey of STDP in Spiking Neural Networks for Pattern Recognition](https://doi.org/10.1109/IJCNN48605.2020.9207239)

[Learning Rules in Spiking Neural Networks: A Survey](http://dx.doi.org/10.1016/j.neucom.2023.02.026)

### ANN-to-SNN

Metoda "ANN-to-SNN" opiera się na **konwertowaniu wcześniej wytrenowanych w klasycznym reżimie sieci** drugiej generacji. Opierają się one na założeniu, że siła aktywacji neuronu II gen. odpowiada częstotliwości aktywacji neuronu pulsującego. Różne **funkcje aktywacji i odmiany warstw ANN muszą zostać zamodelowane przy pomocy neuronów pulsujących**. Przykładowo akceptowalną aproksymacją zachowania funcji ReLU jest neuron Integrate-and-Fire. 

To podejście mierzy się z problemami związanymi z utratą jakości przy konwersji oraz wysokiej gęstości połączeń między neuronami. Aby zaradzić tym problemom stosuje się kilka metod pomocniczych. Po pierwsze, sieci impulsujące można dotrenować w procesie "fine-tuning" przy pomocy innej metody treningu. Inną kwestią jest stworzenie wyspecjalizowanych neuronów lepiej odzwierciedlających funkcje aktywacji. Przykładowo ReLU lepiej odzwierciedla neuron IF ze zmodyfikowaną funkcją resetu (reset przez odejmowanie, a nie reset do zera). 

Ostatnim proponowanym zabiegiem jest poddanie wag sieci pewnej normalizacji przy konwersji.
Zaletą tej metody jest **możliwość szybkiej implementacji znanych już rozwiązań**, jednak takie mogą nie wykorzystywać silnych stron pulsujących sieci neuronowych.

### Surrogate Gradient

Metoda "Surrogate Gradient" jest sposobem na **bezpośrednie trenowanie** sieci pulsujących, jednak **korzystając ze znanych narzędzi** (stochastic gradient descent, backpropagation). Zasadniczo **wymaga to rozwiązania dwóch problemów**: ograniczenia pochodnej w okolicach impulsu neuronu, oraz przetwarzania dynamicznej ewolucji stanu neuronu. 

Drugi z tych problemów ma już swoje rozwiązanie w klasycznej sieci neuronowej, która przechowuje wewnętrzny stan w trakcie przetwarzania - rekurencyjnej sieci neuronowej. W celu trenowania RNN stosuje się algorytm "Backpropagation through time". Algorytm ten jest sprawdzony, chociaż ma swoje wady - w szczególności problem ze zrównoleglaniem treningu.

Problem ewaluacji pochodnej wokół impulsu ma nowe rozwiązanie, od którego pochodzi nazwa metody. Zamiast ewaluowania pochodnej impulsu (która oczywiście dąży do nieskończoności w czasie impulsu) **stosuje się zastępczą pochodną**. Ta pochodna **nie** jest wyprowadzona z funkcji opisującej impuls, jest to po prostu funkcja, której zastosowanie w procesie treningowym daje dobre rezultaty.

Plusem metody jest możliwość bezpośredniego trenowania impulsowych sieci neuronowych znanymi narzędziami. Klasyfikator opisany w dalszej części raportu korzysta z tej właśnie metody.
Niestety ta metoda ma **problem z prędkością zbieżności i ogólnie obniżoną jakością** sieci w porównaniu z klasycznymi odpowiednikami. Wynika to przede wszystkim z prostoty stosowanego modelu neuronu i różnic między podstawionym a rzeczywistym gradientem. Proponowanymi rozwiązaniami są: rozbudowanie neuronu o trenowalny parametr reprezentujący stałą czasową dynamiki neuronu, oraz trenowalny gradient neuronu.

### Inne gradientowe

Istnieją inne mniej popularne metody trenowania w oparciu o algorytm propagacji wstecznej i formalizmu gradientów. Różnią się one przede wszystkim funkcją kosztu. Zamiast częstotliwości **znaczące mogą być chwile czasy impulsu, czy napięcie chwilowe**.

### Spike-Timing-Dependent Plasticity

STPD jest biologicznie inspirowanym (i biologicznie prawdopodobnym) algorytmem treningu opierającym się o prostą **zasadę plastyczności połączeń** między neuronami. Jeżeli impuls przychodzi na wejście neuronu chwilę przed aktywacją to połączenie zostaje wzmocnione. Jeżeli impuls przychodzi na wejście chwilę spóźniony to połączenie zostaje osłabione.
**Metoda ta ma wiele wariantów**, każdy ze swoimi zaletami i problemami. **Główną zaletą** jednak, wspólną dla wszystkich wersji algorytmu, jest **lokalność uczenia**. Trenowanie sieci nie wymaga jak w przypadku propagacji wstecznej wyliczenia przepływu gradientów przez całą sieć, co pozwala na energooszczędną implmentację sprzętową (niskie dystanse poazwalają na niskie napięcia)


## Klasyfikator pulsujący

Klasyfikator został zaimplementowany z pomocą materiałów z ["Training Spiking Neural Networks Using Lessons From Deep Learning"](https://doi.org/10.1109/JPROC.2023.3308088). Jest to **klasyfikator binarny** próbujący wykryć obecność egzoplanet na podstawie fluktuacji jasności obserwowanych gwiazd. Dane wejściowe są szeregiem czasowym.

Do modelowania sieci użyto neuronów typu Leaky Intagrate-and-Fire (LIF). Jest to mniej dokładny model niż omawiany na zajęciach model Izikievitch'a natomiast istnieje więcej sprawdzonych implementacji tego modelu i jest prostszy koncepcyjnie. Do treningu wykorzystano metodę Surrogate Gradient z **sigmoidalną funkcją gradientu** - ma ona podobną charakterystykę jak funkcje wykorzystywane do obliczenia wpływu impulsu w metodach STDP (pomijając nieciągłość)

Jako, że pozytywna klasa jest dla nas znacznie bardziej interesująca, a występuje rzadziej należy zbalansować liczność klas w zbiorze treningowym. W przeciwnym wypadku klasyfikator mógłby osiągać bardzo dobre rezultaty całkowicie ignorując pozytywną klasę. W tym celu wykorzystano metodę wzbogacania danych nazwaną SMOTE. 
Nie wykorzystano zbioru walidacyjnego dlatego nie mamy kontroli nad zjawiskiem przetrenowaniem sieci.

Rezultaty treningu wyglądają rozsądnie, jednak nie będą porównane z sieciami klasycznymi. Celem projektu był przegląd narzędzi, różnic między nimi i ich potencjalnych zastosowań. 