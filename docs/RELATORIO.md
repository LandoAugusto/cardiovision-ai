# CardioVision AI — Relatório de Pré-processamento e Classificação de ECG

**Projeto:** Classificação de Imagens Médicas com Deep Learning  
**Dataset:** ECG Heartbeat Categorization Images (Kaggle — domínio público)

> **PDF:** o relatório formatado para entrega está em [`RELATORIO.pdf`](RELATORIO.pdf).  
> Para regenerar: `python docs/generate_relatorio_pdf.py`

---

## 1. Introdução

Este trabalho implementa um pipeline completo para classificação automática de eletrocardiogramas (ECG) em quatro categorias clínicas: batimentos normais, batimentos anormais, infarto agudo do miocárdio e histórico pós-infarto. O objetivo é aplicar técnicas de visão computacional vistas em aula — pré-processamento, CNN do zero e transfer learning — e avaliar os resultados com métricas padrão.

## 2. Parte 1 — Pré-processamento e Organização

### 2.1 Escolha do dataset

Selecionou-se o dataset **ECG Heartbeat Categorization Images**, por ser público, já organizado por classes e representativo de um problema real de triagem cardíaca por imagem. No projeto local, as classes são: `normal`, `batimento_cardiaco_anormal`, `infarto_do_miocardio` e `historico_pos_infarto` (928 imagens no total).

### 2.2 Etapas do pipeline

| Etapa | Técnica | Justificativa |
|-------|---------|---------------|
| Leitura | OpenCV (`imdecode` + `fromfile`) | Suporte a PNG/JPG e caminhos Unicode no Windows |
| Conversão de cor | BGR → RGB | Padrão esperado por frameworks de DL |
| Redimensionamento | 224 × 224 px | Compatível com VGG16/ResNet50 (ImageNet) |
| Normalização | `pixel / 255 → [0, 1]` | Estabiliza gradientes e acelera convergência |
| Conversão de formato | JPEG (qualidade 95) | Uniformiza armazenamento e I/O |
| Split estratificado | 70% treino / 15% val / 15% teste | Preserva proporção de classes; teste isolado para métricas finais |

A **estratificação** é essencial porque o dataset apresenta desbalanceamento entre classes. A **seed fixa (42)** garante reprodutibilidade entre o notebook de pré-processamento e os de treino.

### 2.3 Estrutura de saída

```
data/processed/
├── train/          # ~70% das imagens
├── validation/     # ~15%
├── test/           # ~15% (usado apenas na avaliação final)
└── metadata.json   # classes, contagens e hiperparâmetros
```

## 3. Parte 2 — Classificação com CNN

### 3.1 Abordagem 1: CNN do zero

Arquitetura sequencial com quatro blocos convolucionais (32→64→128→256 filtros), MaxPooling, camada densa de 512 neurônios, Dropout (0,5) e softmax de 4 saídas. Otimizador Adam (lr=1e-3), loss `sparse_categorical_crossentropy`.

**Callbacks:** EarlyStopping (patience=5) e ModelCheckpoint para salvar o melhor modelo por acurácia de validação.

### 3.2 Abordagem 2: Transfer Learning (ResNet50)

Base **ResNet50** pré-treinada em ImageNet, com camadas congeladas. Entrada passa por `preprocess_input` (normalização específica do ResNet). Camadas finais: GlobalAveragePooling → Dense(256) → Dropout → Dense(4). Learning rate reduzido (1e-4) por se tratar de fine-tuning parcial.

### 3.3 Métricas de avaliação

Todas as métricas são calculadas no **conjunto de teste** (nunca visto no treino):

- **Acurácia** — proporção de acertos globais
- **Matriz de confusão** — erros por classe
- **Precisão, Recall e F1-score** — via `classification_report` do scikit-learn

Os conjuntos de validação e teste são carregados com `shuffle=False` para alinhar labels e predições.

### 3.4 Protótipo de interface

Foi implementado um simulador interativo (`04_prototype_interface.ipynb`) com `ipywidgets`, permitindo selecionar uma imagem de teste e o modelo (CNN ou ResNet50), exibindo a predição e barras de probabilidade.

## 4. Conclusão

O pipeline documentado nos notebooks garante rastreabilidade desde o dado bruto até a inferência. A comparação entre CNN simples e ResNet50 evidencia o ganho típico do transfer learning em datasets médicos de tamanho moderado. Os notebooks estão organizados para execução local ou no Google Colab.

---

*Nota: este sistema é exclusivamente educacional e não deve ser utilizado para diagnóstico clínico.*
