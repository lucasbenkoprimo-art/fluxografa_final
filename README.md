```mermaid
graph TD
    %% Estilos globais
    classDef inicio e fim fill:#f9f,stroke:#333,stroke-width:2px;
    classDef processo fill:#fff,stroke:#333,stroke-width:1px;
    classDef decisao fill:#ffffb3,stroke:#333,stroke-width:1px;
    classDef alerta fill:#ffcccb,stroke:#f00,stroke-width:2px;

    %% --- Bloco de Inicialização (Setup) ---
    Start([Início]) --> Init[Inicializa Serial e Configura Pinos <br> Motor, LED e Buzzer]
    Init --> InitState[Define estados iniciais <br> Alarmes DESLIGADOS]
    InitState --> InitCalc[Lê ADC, calcula Temperatura Inicial <br> e define Pico Recente]
    InitCalc --> InitTimers[Inicializa Timers <br> Respiração e Reset]

    %% --- Bloco Principal (Loop) ---
    InitTimers --> LoopStart[Início do Loop]
    
    LoopStart --> ReadNTC[Lê ADC do NTC e <br> calcula Temperatura Atual em °C]
    
    ReadNTC --> CheckPico{Temperatura C > <br> Pico Recente?}
    
    %% Condição: Temperatura maior que o pico
    CheckPico -- Sim --> UpdatePico[Pico Recente = Temperatura C <br> Atualiza tempo_ultima_respiracao]
    UpdatePico --> CheckAlertaAtivo{Alerta está <br> Ativo? _alerta == 1_}
    CheckAlertaAtivo -- Sim --> DesligaAlarmes[Alerta = 0 <br> Desliga Motor, LED e Buzzer <br> LOG: Respiração detectada]
    CheckAlertaAtivo -- Não --> CalcQueda
    DesligaAlarmes --> CalcQueda
    
    %% Condição: Temperatura menor ou igual ao pico
    CheckPico -- Não --> DecPico[Decresce Pico Recente <br> _pico_recente -= 0.002_]
    DecPico --> CalcQueda
    
    %% Cálculo de queda e exibição
    CalcQueda[Calcula Queda = Pico Recente - Temperatura C <br> Exibe dados no Monitor Serial] --> CheckApneia{Queda > 1.5 E <br> Tempo sem respirar > 10s?}
    
    %% Lógica de Disparo de Alarme
    CheckApneia -- Sim --> CheckAlertaZero{Alerta está <br> Desativado? _alerta == 0_}
    CheckAlertaZero -- Sim --> AtivaAlarmes[Alerta = 1 <br> LIGA Motor, LED e Buzzer <br> LOG: Apneia Detectada]:::alerta
    CheckAlertaZero -- Não --> CheckAlertaLog
    AtivaAlarmes --> CheckAlertaLog
    CheckApneia -- Não --> CheckAlertaLog
    
    %% Log de alerta ativo
    CheckAlertaLog{Alerta == 1?}
    CheckAlertaLog -- Sim --> LogAlerta[LOG: Alerta Ativo!]
    CheckAlertaLog -- Não --> CheckReset
    LogAlerta --> CheckReset
    
    %% Reset Ambiental (60 segundos)
    CheckReset{Tempo desde o último <br> reset > 60s?}
    CheckReset -- Sim --> ResetPico[Pico Recente = Temperatura C <br> Atualiza tempo_ultimo_reset_ambiente]
    CheckReset -- Não --> DelayBlock
    ResetPico --> DelayBlock
    
    %% Finalização do ciclo
    DelayBlock[Aguarda Delay de 150ms] --> LoopEnd([Fim do Loop])
    LoopEnd --> LoopStart
