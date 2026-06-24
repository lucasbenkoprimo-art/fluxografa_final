```mermaid
graph LR
    %% Estilos Monocromáticos (Preto, Cinza e Branco)
    classDef default fill:#ffffff,stroke:#333333,stroke-width:1.5px,color:#333333;
    classDef inicio e fim fill:#f0f0f0,stroke:#333333,stroke-width:2px,color:#333333;
    classDef decisao fill:#ffffff,stroke:#333333,stroke-width:1.5px,color:#333333;
    classDef destaque fill:#1a1a1a,stroke:#000000,stroke-width:2px,color:#ffffff;

    %% --- Fluxo Principal ---
    Start([Início]) --> Setup[Configuração<br>Inicial]
    Setup --> Read[Leitura do<br>Sensor NTC]
    
    %% Lógica de Respiração
    Read --> Cond1{Temperatura<br>Subiu?}
    Cond1 -- Sim --> Resp[Atualiza Pico &<br>Zera Alarmes]
    Cond1 -- Não --> Queda[Reduz Pico<br>Aos Poucos]
    
    %% Lógica de Apneia
    Resp --> Cond2{Queda > 1.5°C<br>por > 10s?}
    Queda --> Cond2
    
    %% Disparo do Alarme (Destaque em Preto)
    Cond2 -- Sim --> Alarme[Dispara Alarmes:<br>Motor, LED e Buzzer]:::destaque
    Cond2 -- Não --> Cond3
    Alarme --> Cond3
    
    %% Reset Ambiental e Fechamento do Loop
    Cond3{Tempo ><br>60s?}
    Cond3 -- Sim --> Reset[Reset<br>Ambiental]
    Cond3 -- Não --> Loop[Delay<br>150ms]
    
    Reset --> Loop
    Loop --> Read
