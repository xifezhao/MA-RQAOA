# MA-RQAOA: Simulations-Framework

Dieses Repository enthält den Python-Code zur Simulation und zum Vergleich des **Majorana-Aware Recursive Quantum Approximate Optimization Algorithm (MA-RQAOA)** mit anderen Ansätzen, wie im Paper *"MA-RQAOA: A System-Algorithm Co-Design Framework for Coherence-Limited Topological Quantum Computing"* beschrieben.

Das Skript simuliert die Lösung von Max-Cut-Problemen auf zufälligen 3-regulären Graphen unter Berücksichtigung der physikalischen Einschränkungen eines simulierten topologischen Quantencomputers (TQC).

## 🚀 Kernkonzepte

Das Framework vergleicht drei verschiedene Algorithmen zur Lösung des Max-Cut-Problems mittels QAOA:

1.  **MA-RQAOA (Proposed)**: Ein Hardware-bewusster Algorithmus, der eine Straffunktion (Penalty) in die klassische Optimierungsschleife integriert. Diese bestraft Parameter, die zu Quantenschaltkreisen führen, deren Ausführungszeit die Kohärenzzeit der Hardware überschreitet. Dies leitet den Optimierer sanft zu physikalisch realisierbaren, qualitativ hochwertigen Lösungen.

2.  **Clipped RQAOA**: Ein einfacherer Hardware-bewusster Ansatz. Er gibt einen unendlichen Kostenwert zurück, sobald ein Parametersatz zu einem nicht ausführbaren Schaltkreis führt. Dies erzeugt eine harte, unüberwindbare Grenze im Optimierungsraum, was den Optimierer in suboptimalen lokalen Minima gefangen halten kann.

3.  **Naive RQAOA**: Ein Hardware-agnostischer Standardansatz. Er optimiert ausschließlich die Energie (Approximationsgüte), ohne die Ausführungszeit zu berücksichtigen. Die physikalische Machbarkeit wird erst nach Abschluss der Optimierung überprüft, was oft zu theoretisch guten, aber praktisch unbrauchbaren Lösungen führt.

### Simuliertes Hardware-Modell

Die Simulation basiert auf einem TQC-Modell mit folgenden temporalen Parametern:

-   **Maximale Kohärenzzeit (`T_MAX`)**: Die maximale Dauer, die ein Quantenzustand aufrechterhalten werden kann.
-   **Klassische Latenz (`T_CLASSICAL`)**: Die Zeit, die für die klassische Verarbeitung zwischen den Quantenoperationen benötigt wird.
-   **Primitive Gate-Zeiten**:
    -   `TAU_B`: Zeit für eine schnelle Braiding-Operation (1-Qubit-Gatter).
    -   `TAU_SWAP`: Zeit für ein benachbartes SWAP-Gatter, um Qubits zu verschieben.
    -   `TAU_F`: Zeit für eine langsame, kostspielige Fusion-Operation (T-Gatter), die hier nicht explizit genutzt, aber im Paper als Motivation erwähnt wird.

Die effektive Zeit, die für den Quantenschaltkreis zur Verfügung steht, ist `COHERENCE_LIMIT = T_MAX - T_CLASSICAL`.

## ⚙️ Features des Codes

-   **Problemgenerierung**: Erzeugt zufällige 3-reguläre Graphen für das Max-Cut-Problem und eine 2D-Gitter-Topologie für die simulierte Hardware.
-   **Quantensimulation mit Qiskit**: Nutzt `qiskit_aer` für eine performante Simulation der QAOA-Schaltkreise.
-   **Exakter Benchmark**: Verwendet die `pulp`-Bibliothek, um die exakte optimale Lösung des Max-Cut-Problems zu berechnen und die Approximationsgüte (`alpha`) zu bestimmen.
-   **Automatisierte Simulation & Analyse**: Führt die Simulation für verschiedene Problemgrößen und QAOA-Tiefen (`p_depth`) durch, speichert die Rohdaten und generiert automatisch die im Paper gezeigten Plots.
-   **Reproduzierbarkeit**: Das Skript ist darauf ausgelegt, die Schlüsselergebnisse des Papers zu reproduzieren, einschließlich:
    -   Approximationsgüte vs. Problemgröße
    -   Machbarkeitsrate (Feasibility Rate) vs. Problemgröße
    -   Optimierungstrajektorie für einen anspruchsvollen Fall

## 📦 Installation

Stellen Sie sicher, dass Sie Python 3.8+ installiert haben. Klonen Sie das Repository und installieren Sie die erforderlichen Abhängigkeiten mit pip:

```bash
pip install numpy networkx scipy matplotlib tqdm pandas pulp qiskit-aer
```

## ▶️ Ausführung

Das Skript kann direkt von der Kommandozeile aus ausgeführt werden. Es führt alle Schritte nacheinander aus: Simulation, Trajektorien-Erfassung und Plot-Erstellung.

```bash
python <skriptname>.py
```

Der Ablauf ist wie folgt:

1.  **`run_and_save_full_simulation()`**: Führt die Hauptsimulation durch. Dies kann je nach Konfiguration (insbesondere `N_RUNS` und `problem_sizes`) einige Zeit in Anspruch nehmen. Die Ergebnisse werden in `multi_p_simulation_results_qiskit1.1.csv` gespeichert.
2.  **`capture_and_save_trajectory()`**: Sucht nach einer "herausfordernden" Probleminstanz (deren Ausführungszeit nahe an der Kohärenzgrenze liegt) und zeichnet den Optimierungspfad für jeden Algorithmus auf. Die Daten werden in `improved_trajectory_results_qiskit1.1.csv` gespeichert.
3.  **`plot_results_from_csv()`**: Liest die beiden CSV-Dateien und generiert eine mehrseitige PDF-Datei (`multi_p_MA_RQAOA_Simulation_Results_Qiskit1.1.pdf`) mit allen relevanten Plots.

## 🔧 Konfiguration

Alle wichtigen Simulations- und Hardwareparameter können am Anfang des Skripts im Abschnitt `1. SIMULIERTE HARDWARE- UND SIMULATIONSPARAMETER` angepasst werden:

```python
# Hardware-Parameter
T_MAX = 1.0
T_CLASSICAL = 0.1
TAU_B = 0.001
TAU_SWAP = 0.003
TAU_F = 0.2

# Simulationsparameter
P_DEPTHS = range(1, 6)      # Zu testende QAOA-Tiefen
PENALTY_MU = 10.0           # Stärke der Straffunktion für MA-RQAOA
N_RUNS = 50                 # Anzahl der Durchläufe pro Konfiguration
OPTIMIZER_MAX_EVALS = 300   # Maximale Auswertungen für den COBYLA-Optimierer

# Pfade für Ausgabedateien
RESULTS_CSV_PATH = "multi_p_simulation_results_qiskit1.1.csv"
TRAJECTORY_CSV_PATH = "improved_trajectory_results_qiskit1.1.csv"
OUTPUT_PDF_PATH = "multi_p_MA_RQAOA_Simulation_Results_Qiskit1.1.pdf"
```

## 📄 Ausgabedateien

Nach einem vollständigen Durchlauf erzeugt das Skript die folgenden Dateien:

-   **`multi_p_simulation_results_qiskit1.1.csv`**: Enthält die detaillierten Ergebnisse jedes einzelnen Simulationslaufs, einschließlich Problemgröße, Algorithmus, Approximationsgüte (`alpha`), Machbarkeit und berechneter Ausführungszeit.
-   **`improved_trajectory_results_qiskit1.1.csv`**: Speichert die von den Optimierern vorgeschlagenen Ausführungszeiten bei jeder Iteration für die spezielle Trajektorien-Analyse.
-   **`multi_p_MA_RQAOA_Simulation_Results_Qiskit1.1.pdf`**: Eine übersichtliche PDF-Datei mit allen Ergebnis-Plots, die zur Analyse und zum Vergleich der Algorithmen dienen.

## 🎓 Zitieren

Wenn Sie diese Arbeit in Ihrer Forschung verwenden, zitieren Sie bitte das Originalpaper:

> Zhao, X., & Wang, H. (2025). MA-RQAOA: A System-Algorithm Co-Design Framework for Coherence-Limited Topological Quantum Computing.
