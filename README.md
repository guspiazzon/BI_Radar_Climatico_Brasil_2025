# BI_Radar_Climatico_Brasil_2025
Um painel climático realizado no Power BI com dados extraídos do Inmet sobre as temperaturas registradas durante todo o ano de 2025 no Brasil, dados foram extraídos e salvos no SQL Server para depois serem mostrados no painel através do Power BI

# 🌡️ Painel Climático Brasileiro 2025 | Análise de Extremos e Amplitude Térmica


> **Projeto de Business Intelligence** focado na análise exploratória de dados meteorológicos do Brasil no ano de 2025. O dashboard identifica picos de temperatura (máximas e mínimas) e calcula a variação da amplitude térmica diária em nível municipal.

---

## 📌 1. Visão Geral & Contexto do Negócio

O ano de 2025 registrou episódios climáticos atípicos em diversas regiões do Brasil. A variação extrema de temperatura afeta diretamente setores como **energia, agricultura, logística e saúde pública**.

O objetivo deste painel é transformar microdados brutos das estações meteorológicas em um **painel executivo de rápida leitura**, permitindo responder a três perguntas centrais:
1. Quais foram as **temperaturas máximas e mínimas absolutas** registradas no país em 2025?
2. Quais cidades apresentaram os **rankings mais quentes e mais frios** do ano?
3. Quais locais sofreram a **maior amplitude térmica no mesmo dia** (variação brusca de temperatura)?

---

## 📸 2. Visão do Dashboard

[Painel Climático 2025]

<img width="1430" height="801" alt="image" src="https://github.com/user-attachments/assets/9469e071-b01b-4a03-9764-14c5d67f5f8c" />



---

## 💡 3. Principais Insights Descobertos

* 🔴 **Pico de Calor:** A maior temperatura do ano foi registrada no **[Rio Grande do Sul], na cidade de [Quarai]** no dia **[04/02/2025]**, atingindo **[43.8 ºC]**.
* 🔵 **Pico de Frio:** A menor temperatura registrada curiosamente foi registrada no mesmo estado foi em **[São José dos Ausentes-RS]** no dia **[03/08/2025]**, marcando **[-9.2 ºC]**.
* ⚡ **Maior Virada de Tempo (Amplitude Diária):** O recorde de variação dentro de um mesmo período de 24 horas aconteceu em **[Rancharia-SP]**, cidade do interior de SP no dia **[11/09/2025]**, com uma diferença de **[30.5 ºC]** entre a mínima e a máxima do dia.

---

## 🛠️ 4. Arquitetura da Solução & Técnicas Utilizadas

### 📊 Fluxo de Dados
1. **Extração & Tratamento:** Limpeza e padronização das bases públicas do INMET (trata campos de data, coordenadas e valores nulos).
2. **Modelagem de Dados:** Criação de estrutura colunar com tabela fato de medições e suporte a consultas performáticas.
3. **Métricas em DAX:** Uso de lógica para busca dinâmica de atributos e cálculos iterativos.

### 🧮 Destaques do Código DAX

#### A. Identificação do Cartão Extremo (Valor + Cidade + Data no mesmo Card)
Para evitar cartões genéricos e trazer contexto no mesmo componente visual, foi utilizada a função `TOPN` concatenada com `UNICHAR(10)` para quebra de linha:

```dax
Card_Maior_Temperatura = 
VAR _MaiorTemp = MAX(stg_inmet_temperaturas[temperatura_maxima])
VAR _TabelaTop = 
    TOPN(
        1, 
        CALCULATETABLE(stg_inmet_temperaturas, stg_inmet_temperaturas[temperatura_maxima] = _MaiorTemp), 
        stg_inmet_temperaturas[data], 
        ASC
    )
VAR _Cidade = SELECTCOLUMNS(_TabelaTop, "Cidade", stg_inmet_temperaturas[cidade])
VAR _Estado = SELECTCOLUMNS(_TabelaTop, "Estado", stg_inmet_temperaturas[estado_abrev])
VAR _DataTexto = SELECTCOLUMNS(_TabelaTop, "Data", stg_inmet_temperaturas[data])
VAR _DataFormatada = FORMAT(CONVERT(_DataTexto, DATETIME), "dd/mm/yyyy")

RETURN
    IF(
        NOT ISBLANK(_MaiorTemp),
        FORMAT(_MaiorTemp, "0.0") & " °C" & UNICHAR(10) & 
        _Cidade & " - " & _Estado & UNICHAR(10) & 
        _DataFormatada,
        "Sem dados"
    )
