sre-copilot-lab/                  <-- Raiz do repositório Git
│
├── docker-compose.yml            <-- Sobe Kafka, MySQL, Zookeeper, Datadog
├── README.md                     <-- A vitrine: Arquitetura, como rodar, prints da IA
├── load-test.js                  <-- Script K6 para injetar milhares de requisições
│
├── gateway-api/                  <-- Microsserviço 1 (Independente)
│   ├── src/
│   ├── pom.xml                   
│   └── Dockerfile                
│
├── payment-processor/            <-- Microsserviço 2 (Independente)
│   ├── src/
│   ├── pom.xml                   
│   └── Dockerfile                
│
└── sre-copilot-agent/            <-- Microsserviço 3 (O cérebro com Gemini)
    ├── src/
    ├── pom.xml                   
    └── Dockerfile
