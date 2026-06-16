# 🏥 Unimed SmartQueue

Sistema inteligente de triagem hospitalar desenvolvido em Java, utilizando o algoritmo de Dijkstra para calcular a melhor rota de atendimento de pacientes dentro de um hospital, considerando nível de urgência, lotação dos setores e tempo médio de atendimento.

## 🚀 Funcionalidades

- Roteamento inteligente com algoritmo de Dijkstra.
- Cálculo dinâmico de custos baseado em:
  - Urgência do paciente;
  - Lotação dos setores;
  - Tempo médio de atendimento.
- Visualização do status dos setores hospitalares.

## 🛠️ Tecnologias

- Java
- Estruturas de Dados (Grafos)
- Algoritmo de Dijkstra
- Priority Queue (`PriorityQueue`)

## 📊 Objetivo

Demonstrar a aplicação prática de grafos e algoritmos de caminho mínimo em um cenário realista de gestão hospitalar, otimizando o fluxo de pacientes conforme critérios clínicos e operacionais.

## ▶️ Como executar

```bash
javac HospitalQueue.java
java HospitalQueue
