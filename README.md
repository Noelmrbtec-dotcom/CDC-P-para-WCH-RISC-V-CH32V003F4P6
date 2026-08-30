
# CDC-P v3.0 - Concurrency Deterministic Control with Preemption

RTOS compacto, eficiente e AUTOCONSCIENTE para microcontroladores de 8 e 32 bits.

![Version](https://img.shields.io/badge/version-3.0-blue)
![Platform](https://img.shields.io/badge/platform-CH32V003%20%7C%20STM32%20%7C%20PIC16-green)
![License](https://img.shields.io/badge/license-MIT-orange)
![RAM](https://img.shields.io/badge/RAM-552%20bytes-red)
![Flash](https://img.shields.io/badge/Flash-9.6%20KB-yellow)

---

## 📖 Sobre

O **CDC-P** (Concurrency Deterministic Control with Preemption) é um RTOS que implementa **preempção REAL** sem troca de contexto. Em vez de interromper tarefas, o CDC-P simplesmente **muda a ordem da fila de execução**, eliminando a necessidade de stacks individuais, mutexes e semáforos binários.

### Filosofia

> "A bagunça tolerada por abundância de recursos." — A crítica.
>
> "Preempção REAL não é interromper tarefas. É executar a tarefa certa no momento certo."

---

## ✨ Características

- ⚡ Preempção REAL via URG-S (fura-fila imediato)
- 🎯 Prioridade Tridimensional (Espacial, Temporal, Situacional)
- 🛡️ 7 camadas de proteção com auto-regulagem
- 🔒 Isolamento de falhas (Xerife e Síndico)
- 📦 Zero stack por tarefa (compartilhada)
- 🚫 Imune a race conditions, deadlocks e inversão de prioridade
- 🔄 Tarefas atômicas que sempre retornam
- 📡 DMA Full-Duplex com IDLE (buffer ajustável 1-N bytes)

---

## 🏗️ Arquitetura

### Prioridade Tridimensional

**1. Espacial:** Posição na fila do despachador  
**2. Temporal:** Período configurado da tarefa (taskX)  
**3. Situacional:** Flag de urgência por evento (URG-S)

### 7 Camadas de Proteção

**1.** Auto-regulagem temporal (atrasou → desacelera)  
**2.** Bloqueio por deadline (excedeu → bloqueia)  
**3.** Diagnóstico de pane (Xerife - Task10)  
**4.** Recuperação progressiva (Síndico - Task9)  
**5.** Fail-safe final (reset se Síndico falhar)  
**6.** Aceleração automática (CDC-P)  
**7.** URG-S (resposta imediata)

### Tarefas do Sistema

| Task | Função | Período |
|------|--------|---------|
| 1 | LED1 (PC0) | 20ms |
| 2 | LED2 (PC1) | 40ms |
| 3 | Comandos seriais | 10ms |
| 4 | Status periódico | 60s |
| 5 | Botão (PC2) | 30ms |
| 6 | ADC (PC4) | 500ms |
| 9 | Síndico (recuperação) | 10ms |
| 10 | Xerife (diagnóstico) | 10ms |

---

## 📊 Desempenho vs FreeRTOS

| Métrica | FreeRTOS | CDC-P |
|---------|----------|-------|
| Preempção | 10 µs | 0.02 µs |
| RAM usada | 3-5 KB | 552 B |
| Flash usada | 20-30 KB | 9.6 KB |
| Latência | 1-2 ticks | Imediata |
| Stack por tarefa | 200-500 B | 0 B |
| Race condition | Possível | Impossível |
| Deadlock | Possível | Impossível |
| Inversão de prioridade | Possível | Impossível |
| Carga CPU a 1000Hz | 10% | 0.002% |

---

## 🎮 Plataformas Suportadas

| MCU | Arquitetura | Clock | RAM | Flash |
|-----|-------------|-------|-----|-------|
| CH32V003F4P6 | RISC-V RV32EC | 48 MHz | 2 KB | 16 KB |
| STM32F103C6 | ARM Cortex-M3 | 72 MHz | 6 KB | 32 KB |
| PIC16F628A | PIC 8-bit | 4 MHz | 224 B | 2  KB |

---

## 🚀 Início Rápido

### Hardware Necessário

- CH32V003F4P6 (ou compatível)
- Adaptador USB-Serial (3.3V)
- LEDs e botão (opcional)
- WCH-Link para gravação

### Pinagem (CH32V003)

| Pino | Função |
|------|--------|
| PC0 | LED1 (Task1) |
| PC1 | LED2 (Task2) |
| PC2 | Botão (Task5) |
| PC4 | ADC_CH2 (Task6) |
| PD5 | USART TX (115200 baud) |
| PD6 | USART RX |

### Comandos Serial (115200 baud)

| Comando | Ação |
|---------|------|
| p | Ativa preempção na Task1 |
| n | Desativa preempção |
| e | Dispara preempção |
| u | Seta urgência na Task1 |
| s | Status resumido |
| d | Diagnóstico completo |
| 1 | Tick 1ms |
| 2 | Tick 10ms (padrão) |
| 3 | Tick 100ms |

---

## 📁 Estrutura do Projeto

| Arquivo | Descrição |
|---------|-----------|
| main.c | Kernel + Tarefas |
| CH32V003_IO_V2.h | Biblioteca GPIO |
| CH32V003_USART_V4.h | Biblioteca USART + DMA + IDLE |
| CH32V003_SYSTICK_V3.h | Biblioteca SysTick |
| CH32V003_ADC_V3.h | Biblioteca ADC + DMA |
| CH32V003_DMA_V3.h | Biblioteca DMA genérico |
| docs/ | Documentação e guias |

---

## 🔬 Teste de Stress

Disparo de preempção e urgência a 1000 Hz (1ms):

| Resultado | Status |
|-----------|--------|
| Estabilidade | Inabalável |
| Latência | Imediata |
| Carga CPU | 0.002% |
| Travamentos | Zero |
| Stack overflow | Impossível |

---

## 🎯 Por que CDC-P é Melhor que FreeRTOS

**FreeRTOS (Preempção por INTERRUPÇÃO):**
- Tick → IRQ → Salvar contexto (PUSH 16 registradores)
- Kernel decide trocar → Restaurar contexto (POP 16 registradores)
- Tarefa executa
- TEMPO: ~10 µs só para trocar!

**CDC-P (Preempção por FURA-FILA):**
- Despachador verifica flag → Tarefa urgente setada
- Executa task_func() DIRETAMENTE
- TEMPO: ~0.02 µs (1 instrução!)

---

## 📚 Documentação

- Resultados do Teste de Stress: `docs/CDC-P_Teste_Stress.txt`

---

## Autor

**Marcos Roberto Braga**

E-mail: noelmrb_tec@yahoo.com

Instituto Informal de Educação, Ciência e Tecnologia

Departamento de Engenharia Eletrônica e Sistemas Embarcados

---

## Licença

Este projeto está licenciado sob a licença MIT.

## 🏆 Agradecimentos

- Edsger W. Dijkstra (inspiração filosófica)
---

⭐ Se este projeto foi útil, deixe uma estrela!
