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

Mehrziel-Fitness:

fitness = A * (num_conflict_edges) + B * (number_of_colors_used - 1) mit A >> B (d.h. Konfliktfreiheit hat Vorrang).

Beispiel: fitness = - (1000 * num_conflicts + 10 * colors_used) (je größer, desto besser; oder negatives Fehlermaß minimieren).

Oder lexikographisch: zuerst num_conflicts minimieren, dann colors_used.

----------------------------------Crossover------------------------------------------------------------------------------------------------------

Uniform Crossover: Für jedes Knoten i, Kind[i] = Eltern1[i] mit p=0.5 sonst Eltern2[i].

One-point / Two-point: auch möglich, aber kann lokale Regionenkohärenz zerstören.

-----------------------------------Mutation----------------------------------------------------------------------------------------------------------

Recolor Mutation: Wähle zufällig einen Knoten und setze colors[i] = random(0..k-1).

Kempe-chain Mutation (fortgeschritten): Austausch zweier Farben entlang einer Komponente , nützlich für Färbungsprobleme.

----------------------------------------Simulated Annealing.........................................................................................

Um die obigen Probleme jeweils mit Simuliertem Annealing (SA) lösen zu können, benötigen Sie zusätzlich zu Kodierung und Fitnessfunktion:

1. Zustandsraum : Dieser ist durch die Kodierung des GA definiert (alle möglichen Platzierungen bzw. Einfärbungen).

2. Startlösung: Eine zufällige Anfangsbelegung, z.B. die erste Dame zufällig platzieren oder die Länder zufällig einfärben.

3. Nachbarschaftsfunktion : Eine Funktion, die aus dem aktuellen Zustand s einen Nachbarzustand s' erzeugt, indem eine kleine lokale Änderung vorgenommen wird.

3.1 8-Damen: Verschieben Sie eine Dame zufällig in eine andere Zeile innerhalb ihrer Spalte oder vertauschen Sie die Zeilenpositionen von zwei zufälligen Spalten.

3.2 Landkarten-Färben: Ändern Sie die Farbe eines zufällig ausgewählten Landes.

4. Kostenfunktion  E(s): Diese entspricht der Fitnessfunktion des GA, aber sie wird typischerweise als zu minimierende Kosten (Energie) formuliert.

# EA.02: Implementierung 


import random
import math
import time
import csv
from collections import defaultdict
from statistics import mean, stdev


# Generic GA utilities

def tournament_selection(pop, fitnesses, k=3):

    best = None
    
    for _ in range(k):
    
        idx = random.randrange(len(pop))
        
        if best is None or fitnesses[idx] > fitnesses[best]:
        
            best = idx
            
    return pop[best]

def pmx_crossover(parent1, parent2):

 
    size = len(parent1)
    
    a, b = sorted(random.sample(range(size), 2))
    
    child1 = [-1]*size
    
    child2 = [-1]*size
    
    child1[a:b+1] = parent1[a:b+1]
    
    child2[a:b+1] = parent2[a:b+1]
    
    def fill_child(child, donor, a, b):
    
        for i in range(a, b+1):
        
            val = child[i]
            
        for i in list(range(0,a)) + list(range(b+1,size)):
        
            v = donor[i]
            
            while v in child[a:b+1]:
            
                # map v via mapping
                
                j = donor.index(child[a + (child[a:b+1].index(v))])
                
                v = parent1[j] if donor is parent2 else parent2[j]
                
            child[i] = v
            
    # simpler PMX implementation using mapping:
    
    def pmx(p1,p2):
    
        size = len(p1)
        
        a,b = a,b
        
        c = [-1]*size
        
        c[a:b+1] = p1[a:b+1]
        
        for i in range(a,b+1):
        
            val = p2[i]
            
        for i in range(size):
        
            if i>=a and i<=b: continue
            
            v = p2[i]
            
            while v in c:
            
                # mapping
                
                idx = p2.index(p1[c.index(v)]) if v in c and False else None
                
                # fallback: linear find not elegant; use iterative map
                
                if v in c:
                
                    # map v by mapping p2[a:b+1] <-> p1[a:b+1]
                    
                    j = p2.index(v)
                    v = p1[j]
                    # repeat until free
                    if v not in c:
                        break
                else:
                    break
            c[i]=v
        # fix any remaining -1 by filling with missing elements
        missing = [x for x in p1 if x not in c]
        for i in range(size):
            if c[i]==-1:
                c[i]=missing.pop(0)
        return c
    # Use simpler: order crossover (OX)
    def ox(p1,p2):
        size = len(p1)
        a,b = a,b
        child = [-1]*size
        child[a:b+1] = p1[a:b+1]
        pos = (b+1)%size
        for i in range(size):
            candidate = p2[(b+1+i)%size]
            if candidate not in child:
                child[pos]=candidate
                pos=(pos+1)%size
        return child
    # We'll return two children using OX (simpler and effective)
    child1 = ox(parent1,parent2)
    child2 = ox(parent2,parent1)
    return child1, child2

def swap_mutation(indiv, prob=0.1):
    """Swap two positions with probability prob (per individual)."""
    if random.random() < prob:
        i,j = random.sample(range(len(indiv)),2)
        indiv[i], indiv[j] = indiv[j], indiv[i]

def uniform_crossover_list(p1, p2):
    size = len(p1)
    child = [None]*size
    for i in range(size):
        child[i] = p1[i] if random.random()<0.5 else p2[i]
    return child


# Problem: 8-Queens

def random_queen_indiv(n=8):
    perm = list(range(n))
    random.shuffle(perm)
    return perm

def queen_conflicts(perm):
    """Count number of conflicting pairs on diagonals."""
    n = len(perm)
    conflicts = 0
    for i in range(n):
        for j in range(i+1,n):
            if abs(perm[i]-perm[j]) == abs(i-j):
                conflicts += 1
    return conflicts

def queen_fitness(perm):
    max_pairs = len(perm)*(len(perm)-1)//2
    return max_pairs - queen_conflicts(perm)  # higher = better


# Problem: Map Coloring (example graph)

def example_map_graph():
    # small example graph: adjacency list (undirected)
    # e.g., map with 9 regions (can be replaced)
    adj = {
        0: [1,2,3],
        1: [0,3,4],
        2: [0,3,5],
        3: [0,1,2,4,5],
        4: [1,3,6],
        5: [2,3,7],
        6: [4,7,8],
        7: [5,6,8],
        8: [6,7]
    }
    return adj

def random_coloring(num_nodes, k_colors):
    return [random.randrange(k_colors) for _ in range(num_nodes)]

def map_conflicts(coloring, adj):
    conflicts = 0
    seen = set()
    for u, neighs in adj.items():
        for v in neighs:
            if (v,u) in seen: continue
            if coloring[u] == coloring[v]:
                conflicts += 1
            seen.add((u,v))
    return conflicts

def map_fitness(coloring, adj):

    num_conf = map_conflicts(coloring, adj)
    
    # also penalize number of colors used (small)
    
    colors_used = len(set(coloring))
    
    return - (1000 * num_conf + 10 * colors_used)  # higher is better when less negative


# GA Engine

def run_ga(problem,

           pop_size=100,
           
           generations=1000,
           
           crossover_rate=0.8,
           
           mutation_rate=0.2,
           
           elitism=1,
           
           tournament_k=3,
           
           stop_on_solution=True):

    # initialize population
    pop = []
    if problem['name']=='queens':
        n = problem.get('n',8)
        for _ in range(pop_size):
            pop.append(random_queen_indiv(n))
        fitness_func = queen_fitness
    elif problem['name']=='map':
        adj = problem['adj']
        k_colors = problem['k_colors']
        for _ in range(pop_size):
            pop.append(random_coloring(len(adj), k_colors))
        fitness_func = lambda indiv: map_fitness(indiv, adj)
    else:
        raise ValueError("Unknown problem")

    best = None
    best_fit = -float('inf')
    history = []
    for gen in range(1, generations+1):
        fitnesses = [fitness_func(ind) for ind in pop]
        # track best
        idx_best = max(range(len(pop)), key=lambda i: fitnesses[i])
        if fitnesses[idx_best] > best_fit:
            best_fit = fitnesses[idx_best]
            best = pop[idx_best].copy()
            best_gen = gen
        history.append(best_fit)

        # check solution condition
        if problem['name']=='queens' and queen_conflicts(best)==0:
            return {'best':best, 'fitness':best_fit, 'gen':gen, 'history':history}
        if problem['name']=='map' and map_conflicts(best, problem['adj'])==0:
            return {'best':best, 'fitness':best_fit, 'gen':gen, 'history':history}

        # new population
        new_pop = []
        # elitism
        sorted_indices = sorted(range(len(pop)), key=lambda i: fitnesses[i], reverse=True)
        for i in range(elitism):
            new_pop.append(pop[sorted_indices[i]].copy())
        # generate remaining
        while len(new_pop) < pop_size:
            parent1 = tournament_selection(pop, fitnesses, k=tournament_k)
            parent2 = tournament_selection(pop, fitnesses, k=tournament_k)
            # crossover
            if random.random() < crossover_rate:
                if problem['name']=='queens':
                    child1, child2 = pmx_crossover(parent1, parent2)
                else:
                    child1 = uniform_crossover_list(parent1, parent2)
                    child2 = uniform_crossover_list(parent2, parent1)
            else:
                child1, child2 = parent1.copy(), parent2.copy()
            #  mutation
            if problem['name']=='queens':
                swap_mutation(child1, prob=mutation_rate)
                swap_mutation(child2, prob=mutation_rate)
            else:
                # per-individual recolor probability
                if random.random() < mutation_rate:
                    i = random.randrange(len(child1))
                    child1[i] = random.randrange(problem['k_colors'])
                if random.random() < mutation_rate:
                    i = random.randrange(len(child2))
                    child2[i] = random.randrange(problem['k_colors'])

            new_pop.append(child1)
            if len(new_pop) < pop_size:
                new_pop.append(child2)
        pop = new_pop
    # finished without perfect solution
    return {'best':best, 'fitness':best_fit, 'gen':generations, 'history':history}
