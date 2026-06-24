```mermaid
graph LR
    %% Estilos visuais limpos e profissionais
    classDef default fill:#f9f9f9,stroke:#333,stroke-width:1px;
    classDef inicio e fim fill:#f4e3f4,stroke:#666,stroke-width:2px;
    classDef decisao fill:#fff9e6,stroke:#888,stroke-width:1px;
    classDef destaque fill:#e6f2ff,stroke:#4a90e2,stroke-width:1px;

    %% --- Inicialização ---
    Start([Início]) --> Setup[Configura Pinos e <br> Define Temperatura Inicial]
    
    %% --- Loop Principal ---
    Setup --> ReadSensor[Lê Sensor NTC e <br> Calcula Temperatura Atual]
    
    %% Lógica de Pico / Respiração
    ReadSensor --> CheckResp{Temperatura subiu? <br> Temp > Pico Recente}
    
    CheckResp -- Sim --> RespDetectada[Atualiza Pico e <br> Desliga Alarmes se ativos]
    CheckResp -- Não --> ReduzPico[Reduz gradativamente <br> o Pico Recente]
    
    %% Lógica de Apneia
    RespDetectada --> CheckApneia{Queda > 1.5°C <br> por mais de 10s?}
    ReduzPico --> CheckApneia
    
    CheckApneia -- Sim --> AtivaAlarmes[Dispara Alarmes: <br> Motor, LED e Buzzer]:::destaque
    CheckApneia -- Não --> CheckReset
    AtivaAlarmes --> CheckReset
    
    %% Reset Ambiental e Loop
    CheckReset{Passou de <br> 60 segundos?}
    CheckReset -- Sim --> ResetAmb[Sincroniza Pico com <br> a Temperatura Atual]
    CheckReset -- Não --> EndLoop
    ResetAmb --> EndLoop
    
    EndLoop[Aguarda 150ms] --> ReadSensor
