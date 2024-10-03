## Simulação RoCEv2

### 1. **Introdução**
- **Motivação**: Inicialmente, planejava-se usar o OMNeT++ para a simulação de RoCE/ RoCEv2. No entanto, o OMNeT++ não possui suporte nativo para esses protocolos, e a implementação seria complexa.
  
### 2. **Alternativa: NS-3 com Suporte para RoCEv2**
- **Repositório Usado**: 
  Utilizamos o repositório: [https://github.com/bobzhuyb/ns3-rdma](https://github.com/bobzhuyb/ns3-rdma), que contém implementações específicas para o RoCEv2 no simulador **NS-3**.
  
- **Ambiente de Desenvolvimento**:
  O projeto foi configurado para ser compilado no **Visual Studio 2015** em um ambiente Windows, devido a questões de compatibilidade com as bibliotecas usadas na época de desenvolvimento.

- **Processo**: 
  1. Realizamos o **build** do projeto no Visual Studio.
  2. Executamos o **main.exe** para rodar a simulação.
  3. A simulação gera arquivos de trace que registram os eventos ocorridos.

### 3. **Explicação do Cenário de Simulação: 2:1 Incast**
Um dos cenários simulados foi o de **"2:1 Incast at 40Gbps for 1 second"**, que representa um padrão típico de tráfego em redes de datacenters.

- **2:1 Incast**: Dois remetentes enviam dados simultaneamente para um único receptor. Esse padrão de tráfego é comum em operações de consulta ou agregação de dados.
- **40Gbps**: A rede entre os remetentes e o receptor foi configurada para operar a uma largura de banda máxima de 40 Gbps.
- **Duração**: O cenário simula a transferência de dados por 1 segundo.

Essa simulação ajuda a estudar **congestionamento de rede**, **controle de fluxo** e **controle de congestionamento** (usando **DCQCN** no RoCEv2).

### 4. **Análise dos Resultados: Trace Gerado**
Ao final da simulação, foi gerado um arquivo **trace** contendo os eventos da rede, incluindo:
- Timestamps
- Tamanho dos pacotes
- Informações de origem e destino dos pacotes

### Análise Detalhada

#### 1. **Timestamp (2.000002, 2.000003, 2.000004)**
   - **O que representa**: Esta é a primeira coluna e mostra o momento exato em que o evento registrado ocorreu, medido em segundos. O timestamp é crucial para entender quando exatamente ocorreu a transmissão de um pacote na simulação.
   - **Exemplo específico**: O primeiro número `2.000002` significa que o evento ocorreu precisamente 2.000002 segundos após o início da simulação.

#### 2. **Origem > Destino (/1 1.2>1.1, /1 1.3>1.1, /1 1.2>1.1)**
   - **O que representa**: A segunda coluna indica a identificação dos nós de origem e destino dos pacotes. O formato `x.y>x.z` indica que o nó com ID `x.y` enviou um pacote para o nó com ID `x.z`.
   - **Exemplo específico**: Em `/1 1.2>1.1`, o nó `1.2` enviou um pacote para o nó `1.1`. Isso pode representar, por exemplo, um servidor (1.2) enviando dados para um cliente (1.1) ou vice-versa, dependendo da configuração da rede simulada.

#### 3. **Protocolo (u)**
   - **O que representa**: A terceira coluna, embora breve aqui com um `u`, geralmente indica o protocolo utilizado na transmissão do pacote. `u` poderia representar UDP, um protocolo comum em simulações que não necessitam de confirmação de recebimento (acknowledgment) dos pacotes.
   - **Exemplo específico**: `u` indica que os pacotes são enviados usando o protocolo UDP.

#### 4. **Tamanho do Pacote (37686, 1740, 9863)**
   - **O que representa**: A quarta coluna mostra o tamanho do pacote enviado, medido em bytes. Este é um dado crucial para análise de desempenho de rede, como throughput e carga na rede.
   - **Exemplo específico**: No primeiro caso, `37686` bytes foram enviados do nó 1.2 para o nó 1.1. Isso pode indicar uma transmissão de dados volumosa, como o envio de um arquivo ou um pacote de dados significativo.

#### 5. **Sequência e Controle (0 3, 0 3, 1 3)**
   - **O que representa**: As últimas colunas podem representar informações adicionais como identificador de sequência e possivelmente flags de controle ou indicadores de fragmentação. 
   - **Exemplo específico**: `0 3` e `1 3` poderiam indicar o número sequencial do pacote e um código de controle ou status, respectivamente.

### Utilidade dessas Informações

As informações contidas em cada linha do arquivo trace permitem aos engenheiros e pesquisadores:
- **Analisar o fluxo de tráfego** entre diferentes nós na rede.
- **Avaliar o desempenho da rede** em termos de throughput, latência e capacidade de carga.
- **Detectar padrões ou problemas** como perda de pacotes, jitter e outras questões de qualidade de serviço (QoS).
- **Simular e ajustar** configurações de rede para otimizar o desempenho e a eficiência.

**Métricas Chave** a serem analisadas:
- **Throughput**: Quantidade de dados transmitidos ao longo do tempo.
- **Latência**: Tempo necessário para os pacotes viajarem do remetente ao receptor.
- **Perda de Pacotes**: Pacotes perdidos durante o congestionamento da rede.

### 5. **Principais Parâmetros da Simulação**
Vamos agora explorar alguns dos principais parâmetros que foram configurados e explicar como eles afetam a simulação e os resultados:

1. **ENABLE_QCN (Yes)**:
   - **Função**: Ativa o **Quantized Congestion Notification (QCN)** para controle de congestionamento.
   - **Importância**: O QCN notifica os remetentes para reduzir a taxa de envio quando há congestionamento, evitando perda de pacotes. Isso melhora o desempenho em situações de tráfego pesado.

2. **USE_DYNAMIC_PFC_THRESHOLD (Yes)**:
   - **Função**: Ativa o uso de limiares dinâmicos para o **Pause Frame Control (PFC)**.
   - **Importância**: Ajusta automaticamente o limite de pausa na transmissão quando os buffers dos switches estão cheios, evitando perdas de pacotes e melhorando a eficiência da rede.

3. **FLOW_LEVEL_ECMP (Yes)**:
   - **Função**: Habilita o **Equal-Cost Multi-Path (ECMP)** para distribuir o tráfego de forma equilibrada em múltiplos caminhos.
   - **Importância**: Reduz a sobrecarga em um único caminho da rede, balanceando melhor a carga e aumentando o throughput.

4. **PACKET_PAYLOAD_SIZE (1000)**:
   - **Função**: Define o tamanho dos pacotes em bytes.
   - **Importância**: Pacotes menores têm menos overhead, enquanto pacotes maiores podem causar mais atraso em situações de congestionamento.

5. **CNP_INTERVAL (50)**:
   - **Função**: Intervalo entre os **Congestion Notification Packets (CNPs)**.
   - **Importância**: O CNP é enviado quando há congestionamento para alertar os remetentes a reduzirem a taxa de envio, evitando congestionamento.

### 6. **Visualização dos Dados do Trace**
O que pode ser feito com os dados do arquivo trace gerado?

#### a. **Gráfico de Throughput ao Longo do Tempo** ///Fizemos
- **Descrição**: Usamos os timestamps e o tamanho dos pacotes transmitidos para calcular o throughput ao longo da simulação.
- **Visualização**: Um gráfico de linha que mostra o throughput variando com o tempo.
  
#### b. **Distribuição do Tamanho dos Pacotes**
- **Descrição**: Plotar um gráfico de barras mostrando a distribuição dos tamanhos dos pacotes transmitidos ao longo da simulação.
  
#### c. **Latência (se aplicável)**
- **Descrição**: Calcular a latência subtraindo o tempo de envio do tempo de recebimento dos pacotes, gerando um gráfico que mostra as variações da latência ao longo do tempo.

### 7. **Resultados**
Ao comparar os dois arquivos **trace** gerados por diferentes execuções da simulação:
- **Diferença de Tamanho dos Pacotes**: Na primeira simulação, os pacotes de **1.2 > 1.1** têm tamanho maior (37686 bytes), enquanto na segunda são menores (9863 bytes).
- **Simetria de Fluxo**: Em ambos os casos, o fluxo é simétrico, com dois nós remetendo pacotes para um único destino, o que cria o cenário de **Incast**.

**Impacto dos Parâmetros no Desempenho**
- **QCN**: O controle de congestionamento via QCN reduziu as perdas de pacotes e evitou que o congestionamento aumentasse.
- **PFC Dinâmico**: O uso de PFC dinâmico ajudou a pausar a transmissão de pacotes quando os buffers estavam cheios, o que reduziu significativamente as perdas de pacotes.

**Alta Frequência de Transmissões**
Os dados mostram uma alta frequência de transmissões entre os nós em um intervalo de tempo extremamente curto (muitos eventos registrados no mesmo segundo, indicado por timestamps como 2.000018, 2.000019, etc.). Isso indica que a rede está processando uma quantidade significativa de dados em uma velocidade muito alta, característica do RoCEv2 que aproveita a eficiência do RDMA para transferências de alta velocidade.

**Consistência no Tamanho dos Pacotes**
Há uma clara consistência nos tamanhos dos pacotes entre dois tipos específicos de transmissões:
   - Nó 1.2 para o nó 1.1: 37686 bytes
   - Nó 1.3 para o nó 1.1: 1740 bytes

Essa consistência no tamanho dos pacotes pode ser um indicativo de que há um padrão de transferência de dados, onde pacotes maiores podem estar transportando dados principais e pacotes menores podem estar carregando sinais de controle ou informações de manutenção da conexão. No contexto do RoCEv2, isso sugere uma eficiência na separação do tráfego de dados de controle do tráfego de dados principal, permitindo otimizações baseadas em prioridade e tipo de tráfego.

**Utilização Eficiente de Priorização de Pacotes**
Se RoCEv2 está configurado para usar o controle de congestionamento ou priorização de pacotes, o tamanho consistente e a frequência dos pacotes podem refletir uma configuração otimizada para garantir que os pacotes mais importantes (dados principais) sejam transmitidos de forma eficaz, mesmo sob cargas pesadas, indicado pelos pacotes de 37686 bytes que são consistentemente enviados entre os nós.

**Possíveis Indicações de Mecanismos de Controle de Fluxo**
A regularidade e a rápida sequência dos timestamps podem também sugerir que os mecanismos de controle de fluxo, como Priority Flow Control (PFC) e Explicit Congestion Notification (ECN), estão ativos para evitar a perda de pacotes. Esses mecanismos são críticos em ambientes RoCEv2 para manter a integridade dos dados e a eficiência da rede, especialmente quando há uma transmissão intensiva como indicado.

**Potencial de Otimização e Ajuste**
Os dados podem ser utilizados para revisar e possivelmente ajustar a configuração da rede. Por exemplo, se encontrarmos momentos onde o desempenho parece decair (não claramente indicado aqui, mas possível em uma análise mais aprofundada), poderíamos ajustar os parâmetros RoCEv2 para balancear melhor a carga, ajustar o tamanho do pacote, ou refinar os limiares de ECN e PFC.

### Conclusão

A análise dos dados do trace fornece uma janela detalhada sobre como os pacotes são gerenciados e transmitidos em uma simulação RoCEv2, destacando a eficácia do RDMA e dos mecanismos associados do RoCEv2 em manter altas taxas de transferência e baixa latência. Essas informações são fundamentais para validar a implementação do RoCEv2, entender seu comportamento em condições operacionais variadas e guiar otimizações futuras para melhorar o desempenho da rede.

- **RoCEv2** se mostrou eficiente na gestão de congestionamento em redes de data centers, especialmente em situações de tráfego intenso como o **Incast**.
- Os parâmetros configurados, como **QCN**, **PFC**, e **ECMP**, ajudam a manter o desempenho e a integridade da rede, evitando a perda de pacotes e mantendo a latência baixa.

---
A SEGUIR: SIMULAÇÃO DO AUTOR DO PROJETO
_________________________________________________________________________________________
# NS-3 simulator for RDMA
This is an NS-3 simulator for RDMA over Converged Ethernet v2 (RoCEv2). It includes the implementation of DCQCN, TIMELY, PFC, ECN and Broadcom shared buffer switch.

It is based on NS-3 version 3.17, and ported to Visual Studio environment, as explained [here](https://www.nsnam.org/wiki/Ns-3_on_Visual_Studio_2012).  

## Note
TIMELY implementation is in "timely" branch and hasn't been merged into the master branch. So, you may not be able to simulate DCQCN and TIMELY simultaneously at this moment.

## Quick Start

### Build
To compile it out-of-the-box, you need Visual Studio *2015* (not 2013 or 2017). 
People have successfully built it with *free* version, 
which can be downloaded [here](https://imagine.microsoft.com/en-us/Catalog/Product/101). 
Open windows/ns-3-dev/ns-3-dev.sln, just build the whole solution.

If you cannot get a Windows machine or Visual Studio for any reason, you may try building it with the original Makefile. We have done it a while back, but now you probably need to edit a few things in waf to make it work.

### Run
The binary will be generated at windows/ns-3-dev/x64/Release/main.exe.
We include a sample configuration file at windows/ns-3-dev/x64/Release/mix/config.txt
Execute main.exe in windows/ns-3-dev/x64/Release/:
```
cd windows\ns-3-dev\x64\Release\
main.exe mix\config.txt
```

It runs a 2:1 incast at 40Gbps for 1 second. Please allow a few minutes for it to finish.
The trace will be generated at mix/mix.tr, as defined by mix/config.txt

There are quite a few options in mix/config.txt. We will gradually add documentation.
For your own convenience you can just check the code, 
project "main" -- source files -- "third.cc", and see how these options are parsed.
You can also raise issues if you have any questions.

## What did we add exactly?

**point-to-point/model/qbb-net-device.cc** and all other qbb-* files: 

DCQCN and PFC implementation.
It also includes go-back-to-N and go-back-to-0 that handle packet drop due to corruption.

In 2013, we got a very basic NS-3 PFC implementation somewhere, and developed based on it. 
We cannot find the original repository anymore.

**network/model/broadcom-node.cc** and **.h**: 

This implements a Broadcom ASIC switch model, which
is mostly doing all kinds of buffer threshold-related operations. These include deciding 
whether PFC should be triggered, ECN should be marked, buffer is too full so packets should
be dropped, etc. It supports both static and dynamic thresholds for PFC.

*Disclaim: this module is purely based on authors' personal understanding of Broadcom ASIC. It does not reflect any official confirmation from either Microsoft or Broadcom.*

**network/utils/broadcom-egress-queue.cc** and **.h**: 

This is the actual MMU buffering packets.
It also includes switch scheduler, i.e., when upper layer ask for a packet to send, it will
decide which queue to be dequeued. Strategies like strict priority and round robin are supported.

**applications/model/udp-echo-client.cc**: 

We implement the RDMA client here, which aligns 
with the fact that RoCEv2 includes UDP header. In particular, original UDP client has troubles
when PFC pause the link. Original UDP client keeps sending packets at line rate, soon
it builds up huge queue and memory runs out. Here we throttle the sending rate if it gets
pushed back by PFC.

**internet/model/seq-ts-header.cc** and **.h**: 

We didn't implement the full InfiniBand
header. Instead, what we really need is just the sequence number (for detecting corruption
drops, and also help us understand the throughput) and timestamp (required by TIMELY.) 
This is where we encode this information into packets.

**main/third.cc**: 

The main() function.

There may be other edits here and there, especially the trace generation is scattered
among various network stacks. But above are the major ones.

## Q&A

**Q: Why do you port it to Windows?**

A: This is a Microsoft project. Visual Studio, including the free version, works well.

**Q: Fine. What if I want to run it on Linux, and do not want to spend time changing the build process?**

A: You can build it using Visual Studio and run the .exe using WINE. We have tested WINE 1.6.2 and it works well.

**Q: I don't understand ... (some part of the code or configuration)**

A: Raise issues on GitHub, so that your questions can also help others. If you really do
not want others know you are working on this, you can email yibzh@microsoft.com

**Q: What papers should I cite, if I also publish?**

A: Below are the ones you should definitely check. They are ranked from most relevant to 
less. That said, all of them are quite relevant:

*ECN or Delay: Lessons Learnt from Analysis of DCQCN and TIMELY*, CoNEXT'16 (this project is released with this paper, we ask you to at least cite this paper if you use this code.)

*Congestion Control for Large-scale RDMA Deployments*, SIGCOMM'15 (DCQCN)

*TIMELY: RTT-based Congestion Control for the Datacenter*, SIGCOMM'15 (TIMELY)

*RDMA over Commodity Ethernet at Scale*, SIGCOMM'16 (discussed go-back-to-N)

*Deadlocks in Datacenter Networks: Why Do They Form, and How to Avoid Them*, HotNets'16 (PFC deadlock analysis, directly used this simulator.)
