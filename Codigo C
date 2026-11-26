#include <stdio.h>

// Definicoes de Firmware / Hardware
#define EXPEDIENTE_INICIO 8
#define EXPEDIENTE_FIM    18

typedef int bit; // Tipo semantico para sinais digitais (0 ou 1)

// Estrutura que representa o barramento de dados da central
typedef struct {
    // 3 Entradas Digitais
    bit in_cartao;      // C: Leitor RFID
    bit in_horario;     // H: Relogio RTC (08h-18h)
    bit in_movimento;   // M: Sensor PIR Interno

    // Sinais Intermediarios
    bit sig_fora_hora;  // NOT H
    bit sig_xor_audit;  // Porta XOR de auditoria

    // Saidas Fisicas
    bit out_trava;      // Saida 1: Rele da Porta Eletrica
    bit out_alarme;     // Saida 2: Sirene / Bloqueio Central
    bit out_log_audit;  // Saida 3: Alerta silencioso no painel do guarda
} CentralSeguranca;

// Nucleo de Processamento Logico (Apenas Portas Booleanas)
CentralSeguranca executarMalhaLogica(bit C, int hora_rtc, bit M) {
    CentralSeguranca sys;

    // 1. Aquisicao das Entradas
    sys.in_cartao    = (C != 0);
    sys.in_horario   = (hora_rtc >= EXPEDIENTE_INICIO && hora_rtc <= EXPEDIENTE_FIM);
    sys.in_movimento = (M != 0);

    // 2. Portas Logicas (NOT, AND, OR, XOR)
    sys.sig_fora_hora = !sys.in_horario;                                      // Porta NOT
    sys.out_trava     = sys.in_cartao && sys.in_horario;                      // Porta AND (Saida 1)
    sys.sig_xor_audit = sys.out_trava ^ sys.in_movimento;                     // Porta XOR

    // Saida 2 (OR + AND + NOT): Dispara sirene se estiver fora de hora E houver (cartao OU movimento)
    sys.out_alarme    = sys.sig_fora_hora && (sys.in_cartao || sys.in_movimento);

    // Saida 3 (AND + XOR): Anomalia operacional durante o expediente
    sys.out_log_audit = sys.in_horario && sys.sig_xor_audit;

    return sys;
}

void renderizarPainelCorporativo(CentralSeguranca s, int hora_rtc) {
    printf("\n+-----------------------------------------------------------------------+\n");
    printf("|  NEXUS ACCESS CONTROL v2.4  |  TELEMETRIA DE HARDWARE EM TEMPO REAL   |\n");
    printf("+-----------------------------------------------------------------------+\n");
    printf("| [ENTRADAS]  RFID (C): %d   |   RTC (H): %d [%02d:00h]   |   PIR (M): %d    |\n",
           s.in_cartao, s.in_horario, hora_rtc, s.in_movimento);
    printf("+-----------------------------------------------------------------------+\n");
    printf("| [MALHA LOGICA]                                                        |\n");
    printf("|   > AND  (C && H)             = %d  -> Rele da Porta Eletrica          |\n", s.out_trava);
    printf("|   > NOT  (!H)                 = %d  -> Flag Fora de Expediente         |\n", s.sig_fora_hora);
    printf("|   > XOR  (Trava ^ M)          = %d  -> Divergencia Fisica              |\n", s.sig_xor_audit);
    printf("|   > OR   (!H && (C || M))     = %d  -> Acionamento de Sirene           |\n", s.out_alarme);
    printf("+-----------------------------------------------------------------------+\n");
    printf("| [SAIDAS FISICAS]                                                      |\n");
    printf("|   SAIDA 1 [TRAVA ELETRICA] : %d  [ %-30s ]  |\n",
           s.out_trava, s.out_trava ? "ENERGIZADA - PORTA ABERTA" : "BLOQUEADA - PORTA TRANCADA");
    printf("|   SAIDA 2 [SIRENE ALARME]  : %d  [ %-30s ]  |\n",
           s.out_alarme, s.out_alarme ? "DISPARADA - INTRUSAO/BLOQUEIO" : "INATIVA - SISTEMA ARMADO");
    printf("|   SAIDA 3 [PAINEL GUARDA]  : %d  [ %-30s ]  |\n",
           s.out_log_audit, s.out_log_audit ? "LOG DE AUDITORIA GERADO" : "OPERACAO PADRAO");
    printf("+-----------------------------------------------------------------------+\n");

    // Analise Semantica da Central
    printf("\n[DIAGNOSTICO DA CENTRAL]: ");
    if (s.out_trava && s.in_movimento) {
        printf("ACESSO AUTORIZADO E CONFIRMADO. Colaborador validou RFID e adentrou o setor.\n");
    } 
    else if (s.out_log_audit && s.out_trava && !s.in_movimento) {
        printf("ALERTA XOR (TIMEOUT): Credencial aceita, porem sensor PIR nao detectou passagem. Porta retrancada.\n");
    } 
    else if (s.out_log_audit && !s.out_trava && s.in_movimento) {
        printf("ALERTA XOR (PRESENCA SEM CREDENCIAL): Movimento interno em horario comercial sem leitura recente na porta.\n");
    } 
    else if (s.out_alarme && s.in_cartao && s.in_movimento) {
        printf("VIOLACAO CRITICA: Tentativa de acesso fora de horario COM movimento interno simultaneo!\n");
    } 
    else if (s.out_alarme && s.in_cartao) {
        printf("BLOQUEIO DE PERIODO: Credencial valida rejeitada pelo relogio RTC (%02dh fora de 08h-18h).\n", hora_rtc);
    } 
    else if (s.out_alarme && s.in_movimento) {
        printf("INTRUSAO PERIMETRAL: Movimento detectado na area restrita sem abertura de porta fora do expediente!\n");
    } 
    else {
        printf("STANDBY: Perimetro seguro, sem eventos ativos.\n");
    }
}

int main() {
    int continuar, cartao, hora, movimento;

    do {
        printf("\n=========================================================================\n");
        printf("   NEXUS SECURITY SYSTEMS - MODULO DE SIMULACAO DE CONTROLE DE ACESSO\n");
        printf("=========================================================================\n");

        do {
            printf(" > Leitor RFID - Cartao Autorizado? (1 = Sim / 0 = Nao): ");
            scanf("%d", &cartao);
        } while (cartao != 0 && cartao != 1);

        do {
            printf(" > Relogio RTC - Hora do Evento (0 a 23): ");
            scanf("%d", &hora);
        } while (hora < 0 || hora > 23);

        do {
            printf(" > Sensor PIR  - Movimento Interno? (1 = Sim / 0 = Nao): ");
            scanf("%d", &movimento);
        } while (movimento != 0 && movimento != 1);

        CentralSeguranca resultado = executarMalhaLogica(cartao, hora, movimento);
        renderizarPainelCorporativo(resultado, hora);

        printf("\nExecutar novo ciclo de teste? (1 = Sim / 0 = Encerrar): ");
        scanf("%d", &continuar);

    } while (continuar == 1);

    printf("\n[SISTEMA ENCERRADO] Sessao de telemetria finalizada com sucesso.\n");
    return 0;
}
