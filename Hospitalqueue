import java.util.*;

/**
 * =====================================================================
 *  Unimed SmartQueue -- Triagem Inteligente com Algoritmo de Dijkstra
 * =====================================================================
 */
public class HospitalQueue {

    static final String RESET  = "\u001B[0m";
    static final String BOLD   = "\u001B[1m";
    static final String RED    = "\u001B[31m";
    static final String YELLOW = "\u001B[33m";
    static final String GREEN  = "\u001B[32m";
    static final String BLUE   = "\u001B[34m";
    static final String CYAN   = "\u001B[36m";

    enum Prioridade {
        VERMELHO("VERMELHO", 1, "Emergencia -- atendimento imediato"),
        LARANJA ("LARANJA",  2, "Muito Urgente -- ate 10 min"),
        AMARELO ("AMARELO",  3, "Urgente -- ate 60 min"),
        VERDE   ("VERDE",    4, "Pouco Urgente -- ate 120 min"),
        AZUL    ("AZUL",     5, "Nao Urgente -- ate 240 min");

        final String label;
        final int nivel;
        final String descricao;

        Prioridade(String label, int nivel, String descricao) {
            this.label = label;
            this.nivel = nivel;
            this.descricao = descricao;
        }
    }

    static class Setor {
        final String id, nome;
        int capacidadeMax, ocupacaoAtual, tempoMedioAtendimento;

        Setor(String id, String nome, int cap, int ocup, int tempo) {
            this.id = id; this.nome = nome;
            this.capacidadeMax = cap; this.ocupacaoAtual = ocup;
            this.tempoMedioAtendimento = tempo;
        }

        double disponibilidade() {
            return 1.0 - ((double) ocupacaoAtual / capacidadeMax);
        }

        String statusBar() {
            int pct = (int)((double) ocupacaoAtual / capacidadeMax * 10);
            String cor = pct >= 9 ? RED : pct >= 6 ? YELLOW : GREEN;
            return cor + "#".repeat(pct) + ".".repeat(10 - pct) + RESET
                + String.format(" %d/%d", ocupacaoAtual, capacidadeMax);
        }
    }

    static class Aresta {
        final String destino;
        final double pesoBase;
        Aresta(String destino, double pesoBase) {
            this.destino = destino; this.pesoBase = pesoBase;
        }
    }

    static class No implements Comparable<No> {
        final String setorId;
        final double custo;
        final List<String> caminho;

        No(String setorId, double custo, List<String> caminho) {
            this.setorId = setorId; this.custo = custo;
            this.caminho = new ArrayList<>(caminho);
        }

        @Override
        public int compareTo(No outro) {
            return Double.compare(this.custo, outro.custo);
        }
    }

    static class GrafoHospitalar {
        final Map<String, Setor> setores = new LinkedHashMap<>();
        final Map<String, List<Aresta>> adjacencias = new HashMap<>();

        void adicionarSetor(Setor s) {
            setores.put(s.id, s);
            adjacencias.putIfAbsent(s.id, new ArrayList<>());
        }

        void adicionarAresta(String origem, String destino, double pesoBase) {
            adjacencias.get(origem).add(new Aresta(destino, pesoBase));
        }

        /**
         * Custo dinamico: combina tempo de espera, lotacao e urgencia.
         *
         * Regras clinicas por protocolo Manchester:
         *
         * VERMELHO/LARANJA (nivel 1-2): risco de vida
         *   - Obrigatorio: Emergencia e Sala Vermelha
         *   - Bloqueado: Ortopedia, Clinica
         *
         * AMARELO (nivel 3): urgente, trauma/fratura
         *   - Rota: Triagem -> Ortopedia -> Imagem -> Observacao -> Alta
         *   - Bloqueado: Emergencia, Sala Vermelha, Clinica
         *   - Alta so permitida apos Imagem
         *
         * VERDE/AZUL (nivel 4-5): nao urgente, sintomas leves
         *   - Rota: Triagem -> Clinica -> (Exames) -> Alta
         *   - Bloqueado: Emergencia, Sala Vermelha, Ortopedia, Internacao
         */
        double calcularCusto(Aresta aresta, Prioridade prioridade) {
            Setor destino = setores.get(aresta.destino);
            if (destino == null) return Double.MAX_VALUE;

            // VERMELHO / LARANJA: somente via Emergencia
            if (prioridade.nivel <= 2) {
                if (aresta.destino.equals("ORTOPEDIA"))     return Double.MAX_VALUE;
                if (aresta.destino.equals("CLINICA"))       return Double.MAX_VALUE;
            }

            // AMARELO: trauma/fratura, protocolo exige Imagem antes de qualquer saida
            if (prioridade.nivel == 3) {
                if (aresta.destino.equals("EMG"))           return Double.MAX_VALUE;
                if (aresta.destino.equals("SALA_VERMELHA")) return Double.MAX_VALUE;
                if (aresta.destino.equals("CLINICA"))       return Double.MAX_VALUE;
                if (aresta.destino.equals("INTERNACAO"))    return Double.MAX_VALUE;
                // Saindo da Ortopedia, so pode ir para Imagem
                // Observacao e Alta diretas sao bloqueadas (peso 10 e 20 respectivamente)
                if (aresta.destino.equals("OBSERVACAO") && aresta.pesoBase == 10)
                    return Double.MAX_VALUE;
                if (aresta.destino.equals("ALTA") && aresta.pesoBase == 20)
                    return Double.MAX_VALUE;
            }

            // VERDE / AZUL: sintomas leves, somente via Clinica
            if (prioridade.nivel >= 4) {
                if (aresta.destino.equals("EMG"))           return Double.MAX_VALUE;
                if (aresta.destino.equals("SALA_VERMELHA")) return Double.MAX_VALUE;
                if (aresta.destino.equals("ORTOPEDIA"))     return Double.MAX_VALUE;
                if (aresta.destino.equals("INTERNACAO"))    return Double.MAX_VALUE;
            }

            double fatorLotacao  = 1.0 + (1.0 - destino.disponibilidade()) * 2.0;
            double fatorUrgencia = 1.0 / (6.0 - prioridade.nivel);
            double tempoEspera   = destino.tempoMedioAtendimento * fatorLotacao;

            return (aresta.pesoBase + tempoEspera) * fatorUrgencia;
        }

        /**
         * Dijkstra -- caminho de menor custo de 'origem' ate 'destino'
         */
        ResultadoDijkstra dijkstra(String origem, String destino, Prioridade prioridade) {
            Map<String, Double> distancias = new HashMap<>();
            setores.keySet().forEach(id -> distancias.put(id, Double.MAX_VALUE));
            distancias.put(origem, 0.0);

            PriorityQueue<No> fila = new PriorityQueue<>();
            fila.add(new No(origem, 0.0, List.of(origem)));

            int totalIteracoes = 0;

            while (!fila.isEmpty()) {
                No atual = fila.poll();
                totalIteracoes++;

                if (atual.setorId.equals(destino)) {
                    return new ResultadoDijkstra(atual.caminho, atual.custo, totalIteracoes);
                }

                if (atual.custo > distancias.get(atual.setorId)) continue;

                for (Aresta aresta : adjacencias.getOrDefault(atual.setorId, List.of())) {
                    double novoCusto = atual.custo + calcularCusto(aresta, prioridade);
                    if (novoCusto < distancias.getOrDefault(aresta.destino, Double.MAX_VALUE)) {
                        distancias.put(aresta.destino, novoCusto);
                        List<String> novoCaminho = new ArrayList<>(atual.caminho);
                        novoCaminho.add(aresta.destino);
                        fila.add(new No(aresta.destino, novoCusto, novoCaminho));
                    }
                }
            }

            return new ResultadoDijkstra(List.of(), Double.MAX_VALUE, totalIteracoes);
        }
    }

    record ResultadoDijkstra(List<String> caminho, double custoTotal, int iteracoes) {}

    static GrafoHospitalar construirHospitalUnimed() {
        GrafoHospitalar g = new GrafoHospitalar();

        g.adicionarSetor(new Setor("RECEPCAO",     "Recepcao & Cadastro",      10,  7, 10));
        g.adicionarSetor(new Setor("TRIAGEM",      "Triagem Manchester",         5,  4, 15));
        g.adicionarSetor(new Setor("EMG",          "Emergencia / Reanimacao",    3,  2,  5));
        g.adicionarSetor(new Setor("SALA_VERMELHA","Sala Vermelha",              2,  1, 30));
        g.adicionarSetor(new Setor("CLINICA",      "Clinica Medica",            12,  9, 40));
        g.adicionarSetor(new Setor("ORTOPEDIA",    "Ortopedia & Trauma",         6,  3, 45));
        g.adicionarSetor(new Setor("EXAMES_LAB",   "Exames Laboratoriais",       8,  6, 25));
        g.adicionarSetor(new Setor("EXAMES_IMG",   "Imagem (RX/TC/USG)",         4,  3, 35));
        g.adicionarSetor(new Setor("OBSERVACAO",   "Observacao / Maca",         15, 11, 60));
        g.adicionarSetor(new Setor("INTERNACAO",   "Internacao",                20, 14, 480));
        g.adicionarSetor(new Setor("ALTA",         "Alta / Liberacao",          99,  0,  5));

        g.adicionarAresta("RECEPCAO",     "TRIAGEM",        2);
        g.adicionarAresta("TRIAGEM",      "EMG",            1);
        g.adicionarAresta("TRIAGEM",      "CLINICA",        3);
        g.adicionarAresta("TRIAGEM",      "ORTOPEDIA",      3);
        g.adicionarAresta("EMG",          "SALA_VERMELHA",  1);
        g.adicionarAresta("EMG",          "EXAMES_LAB",     5);
        g.adicionarAresta("EMG",          "EXAMES_IMG",     5);
        g.adicionarAresta("SALA_VERMELHA","OBSERVACAO",     5);
        g.adicionarAresta("SALA_VERMELHA","INTERNACAO",    10);
        g.adicionarAresta("CLINICA",      "EXAMES_LAB",     5);
        g.adicionarAresta("CLINICA",      "EXAMES_IMG",     5);
        g.adicionarAresta("CLINICA",      "OBSERVACAO",    10);
        g.adicionarAresta("CLINICA",      "ALTA",          15);
        g.adicionarAresta("ORTOPEDIA",    "EXAMES_IMG",     5);
        g.adicionarAresta("ORTOPEDIA",    "OBSERVACAO",    10);
        g.adicionarAresta("ORTOPEDIA",    "INTERNACAO",    20);
        g.adicionarAresta("ORTOPEDIA",    "ALTA",          20);
        g.adicionarAresta("EXAMES_LAB",   "CLINICA",        5);
        g.adicionarAresta("EXAMES_LAB",   "OBSERVACAO",    10);
        g.adicionarAresta("EXAMES_IMG",   "CLINICA",        5);
        g.adicionarAresta("EXAMES_IMG",   "ORTOPEDIA",      5);
        g.adicionarAresta("EXAMES_IMG",   "OBSERVACAO",    10);
        g.adicionarAresta("OBSERVACAO",   "INTERNACAO",    30);
        g.adicionarAresta("OBSERVACAO",   "ALTA",          30);
        g.adicionarAresta("INTERNACAO",   "ALTA",         480);

        return g;
    }

    static void printLinha() {
        System.out.println(BLUE + "=".repeat(62) + RESET);
    }

    static void printSubLinha() {
        System.out.println(CYAN + "-".repeat(62) + RESET);
    }

    static void printCabecalho() {
        printLinha();
        System.out.println(BOLD + BLUE +
            "  [+] UNIMED SmartQueue - Roteamento por Dijkstra" + RESET);
        System.out.println(CYAN +
            "      Sistema Inteligente de Triagem Hospitalar" + RESET);
        printLinha();
    }

    static void printStatusHospital(GrafoHospitalar g) {
        System.out.println(BOLD + "\n[STATUS] SETORES DO HOSPITAL\n" + RESET);
        System.out.printf("  %-24s %-26s %s%n", "Setor", "Ocupacao", "Tempo Medio");
        printSubLinha();
        for (Setor s : g.setores.values()) {
            if (s.id.equals("ALTA")) continue;
            System.out.printf("  %-24s %s  %s%d min%s%n",
                s.nome, s.statusBar(), YELLOW, s.tempoMedioAtendimento, RESET);
        }
        System.out.println();
    }

    static void printResultado(GrafoHospitalar g, ResultadoDijkstra res,
                                String paciente, Prioridade prioridade) {
        printLinha();
        System.out.printf("%n%sPACIENTE:%s %s%n", BOLD, RESET, paciente);
        System.out.printf("%sPRIORIDADE:%s %s -- %s%n%n",
            BOLD, RESET, prioridade.label, prioridade.descricao);

        if (res.caminho().isEmpty()) {
            System.out.println(RED + "  [ERRO] Nenhum caminho encontrado!" + RESET);
            return;
        }

        System.out.println(BOLD + ">> CAMINHO OTIMO CALCULADO:" + RESET);
        System.out.println();

        for (int i = 0; i < res.caminho().size(); i++) {
            String id = res.caminho().get(i);
            Setor s = g.setores.get(id);
            String seta = (i < res.caminho().size() - 1) ? " ->" : "";
            String cor = id.equals("ALTA") ? GREEN :
                         id.equals("EMG") || id.equals("SALA_VERMELHA") ? RED : CYAN;
            System.out.printf("  %s[%d] %-24s%s%s%n",
                cor + BOLD, i + 1, s.nome, RESET, seta);
        }

        System.out.printf("%n%sCusto total estimado:%s %.1f unidades%n",
            BOLD, RESET, res.custoTotal());
        System.out.printf("%sIteracoes do Dijkstra:%s %d nos processados%n",
            BOLD, RESET, res.iteracoes());
        System.out.println();
    }

    public static void main(String[] args) {
        printCabecalho();

        GrafoHospitalar hospital = construirHospitalUnimed();
        printStatusHospital(hospital);

        printResultado(hospital,
            hospital.dijkstra("RECEPCAO", "ALTA", Prioridade.VERMELHO),
            "Joao Silva, 58 anos - Dor toracica intensa",
            Prioridade.VERMELHO);

        printResultado(hospital,
            hospital.dijkstra("RECEPCAO", "ALTA", Prioridade.AMARELO),
            "Maria Souza, 34 anos - Suspeita de fratura no pulso",
            Prioridade.AMARELO);

        printResultado(hospital,
            hospital.dijkstra("RECEPCAO", "ALTA", Prioridade.VERDE),
            "Pedro Lima, 22 anos - Dor de garganta ha 2 dias",
            Prioridade.VERDE);

        printLinha();
        System.out.println(BOLD + "\n[COMPARATIVO] CUSTO POR NIVEL DE URGENCIA\n" + RESET);
        System.out.printf("  %-12s %-14s %s%n", "Prioridade", "Custo Total", "Iteracoes");
        printSubLinha();
        for (Prioridade p : Prioridade.values()) {
            var r = hospital.dijkstra("RECEPCAO", "ALTA", p);
            String cor = p == Prioridade.VERMELHO ? RED :
                         p == Prioridade.LARANJA  ? "\u001B[91m" :
                         p == Prioridade.AMARELO  ? YELLOW :
                         p == Prioridade.VERDE    ? GREEN  : BLUE;
            System.out.printf("  %s%-12s%s %-14.1f %d nos%n",
                cor, p.label, RESET, r.custoTotal(), r.iteracoes());
        }
        printLinha();
        System.out.printf("%n%s[OK] SmartQueue ativo - Todos os pacientes roteados!%s%n%n",
            GREEN + BOLD, RESET);
    }
}
