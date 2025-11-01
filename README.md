# KI_Blatt2

# EA.01: Modellierung von GA:

# 8-Queens-Problem

Das Ziel ist es, acht Damen so auf einem 8 X 8 Schachbrett zu platzieren, dass sich keine gegenseitig schlagen.

Kodierung der IndividuenKodierung: 

Ein Array der Länge 8, wobei jede Position (Index i gehört zu {1, ..., 8}) die Zeile repräsentiert, in der die Dame in der i-ten Spalte steht. Die Werte (Allele) sind aus {1,..., 8}.

Beispiel: Chromosom = [4, 6, 8, 2, 7, 5, 1, 3]  bedeutet, dass die Dame in Spalte 1 in Zeile 4, die Dame in Spalte 2 in Zeile 6, usw. steht.

Die gewählte Kodierung als Array der Zeilenpositionen (z.B. [4, 6, 8, 2, ..]) stellt sicher, dass jede Spalte eine Dame enthält und, wenn die Werte eine Permutation von 
{1, .., 8} bilden, auch jede Zeile nur einmal belegt ist. Dies reduziert den Suchraum signifikant.

----------------------------------Fitnessfunktion--------------------------------------------------

Ziel: minimieren Diagonalkonflikte. Verwende z.B.:

fitness = max_pairs - number_of_conflicting_pairs

Für 8 Damen: max_pairs = 8*(8-1)/2 = 28. Oder setze fitness = 28 - conflicts (je höher desto besser).

Begründung: Die Fitnessfunktion misst, wie nah eine Platzierung an der Konfliktfreiheit ist, indem sie die Anzahl der Verstöße (Schlagbeziehungen auf Diagonalen) direkt bewertet.

--------------------------------Crossover-Operatoren-------------------------------------------------------------

Order Crossover (OX) oder Partially Mapped Crossover (PMX) 

Diese Operatoren sind für Permutationen konzipiert und stellen sicher, dass alle Ziffern (Zeilenpositionen) einmal vorkommen und das Kind somit gültige Zeilenpositionen hat

Beispiel: PMX zwischen zwei Eltern → zwei gültige Kinder ohne Duplikate.

------------------------------------Mutation --------------------------------------------------------------------

Swap-Mutation: Wähle zwei Positionen und tausche deren Werte (Reihen). Das behält Permutationseigenschaft.

Alternativen: Scramble (kleines Segment zufällig permutieren).

Selektion / Elterwahl

Tournament Selection (k=2..5) oder Roulette Wheel, Elitismus (z.B. Top 1 bleibt erhalten).

Begründung: Spezielle Permutations-Operatoren sind erforderlich, um die Gültigkeit der Individuen (jede Zeile nur einmal belegt) zu erhalten.

# Landkarten-Färbeproblem

------------------------------------Kodierung-------------------------------------------------------------------------------------
Ein Array der Länge N (Anzahl der Länder), wobei jeder Index einem Land entspricht und der Wert (Allel) die Farbe des Landes aus der Menge der verfügbaren Farben (z.B. {1, 2, 3, 4, 5) angib)t.Beispiel: Chromosom = [1, 2, 1, 3, 2, 1]

Begründung: Diese Kodierung ist direkt und einfach, da jeder Genort einem Land und der Allelwert einer Farbe entspricht.

------------------------------------Fitnessfunktion---------------------------------------------------------------------------------

Ziel: Konflikte minimieren, Farbenanzahl minimieren.

Mehrziel-Fitness (zusammengefaßt):

fitness = A * (num_conflict_edges) + B * (number_of_colors_used - 1) mit A >> B (d.h. Konfliktfreiheit hat Vorrang).

Beispiel: fitness = - (1000 * num_conflicts + 10 * colors_used) (je größer, desto besser; oder negatives Fehlermaß minimieren).

Oder lexikographisch: zuerst num_conflicts minimieren, dann colors_used.

----------------------------------Crossover------------------------------------------------------------------------------------------------------

Uniform Crossover: Für jedes Knoten i, Kind[i] = Eltern1[i] mit p=0.5 sonst Eltern2[i].

One-point / Two-point: auch möglich, aber kann lokale Regionenkohärenz zerstören.

-----------------------------------Mutation----------------------------------------------------------------------------------------------------------

Recolor Mutation: Wähle zufällig einen Knoten und setze colors[i] = random(0..k-1).

Kempe-chain Mutation (fortgeschritten): Austausch zweier Farben entlang einer Komponente , nützlich für Färbungsprobleme.
