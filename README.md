# 📊 Fase 3 – 1TIAOS – Cap 1 - Etapas de uma Máquina Agrícola
## **Importação e Análise de Dados de Sensores no Oracle Database**

### 👨‍💻 Aluno:
- [Silvio Prestes Guerreiro Junior](https://www.linkedin.com/in/silvio-guerreiro-junior/)  
- **Matrícula:** RM567958  

### 👩‍🏫 Professores:
- **Tutor(a):** Sabrina Otoni  
- **Coordenador(a):** André Godoi Chiovato  

---

## 📡 Visão Geral do Projeto

Este projeto foi desenvolvido como parte da **Fase 3 do curso 1TIAOS – FIAP**, no contexto do **Capítulo 1: Etapas de uma Máquina Agrícola**.  
O objetivo principal foi **importar, tratar e analisar dados de sensores agrícolas** no **Oracle Database**, garantindo a integridade e a coerência dos valores originais provenientes da simulação prática da **Fase 2 – Capítulo 1 do projeto Farm Tech Solutions**.

Os dados representam medições de sensores de **pH**, **ajuste de pH**, **umidade relativa**, **temperatura** e **nutrientes (N, P, K)** — variáveis fundamentais para o controle automatizado de irrigação e monitoramento ambiental em lavouras inteligentes.

---

## 🧠 Origem dos Dados

As informações utilizadas neste projeto foram **copiadas diretamente do terminal do Visual Studio Code (VS Code)**, **durante a execução da solução implementada na Fase 2 – Capítulo 1 do projeto *Farm Tech Solutions***, utilizando o **simulador Wokwi** (com microcontrolador ESP32, sensor DHT22, sensor de pH e módulo relé).

Após a simulação e coleta, os dados foram:
1. Copiados do **terminal do VS Code** durante a execução da simulação;  
2. Salvos em uma **planilha Excel (`Sensores_limpo.xlsx`)**;  
3. Importados e tratados no **Oracle SQL Developer**, conforme solicitado na atividade prática.

Exemplo de leituras registradas no terminal:

```

N=0 P=0 K=0 | pH=4.41 | Ajuste pH=0.0 | Umi=40.0% | Temp=24.8C | Bomba=OFF -> pH fora (alvo 5.5–6.8)
N=0 P=0 K=0 | pH=5.55 | Ajuste pH=0.0 | Umi=40.0% | Temp=24.8C | Bomba=OFF -> aguarda: Umi <55% e Temp >=40C

````

Esses dados simulam as condições ambientais monitoradas por sensores em campo e foram posteriormente utilizados como base para a importação e análise dentro do banco Oracle.

---

## ⚙️ Objetivo

- **Preservar a precisão decimal** dos dados originais durante a importação;  
- **Evitar erros de conversão e precisão** (como `ORA-00957` e `ORA-01438`);  
- **Garantir que os tipos de dados** definidos na planilha fossem refletidos corretamente no banco Oracle;  
- **Realizar consultas SQL analíticas** sobre o conjunto importado.

---

## 🧱 Estrutura da Tabela `SENSORES`

### 📄 Criação da tabela
```sql
CREATE TABLE SENSORES (
  N              NUMBER(1),
  P              NUMBER(1),
  K              NUMBER(1),
  PH             NUMBER(3,2)     NOT NULL,
  AJUSTE_PH      NUMBER(3,2),
  UMIDADE_       NUMBER(3,1),
  TEMPERATURA_C  NUMBER(3,1),
  MENSAGEM       VARCHAR2(77 CHAR)
);
````

### 🔧 Ajustes de Precisão e Escala

```sql
ALTER TABLE SENSORES MODIFY (PH             NUMBER(4,2));
ALTER TABLE SENSORES MODIFY (AJUSTE_PH      NUMBER(4,2));
ALTER TABLE SENSORES MODIFY (UMIDADE_       NUMBER(5,2));
ALTER TABLE SENSORES MODIFY (TEMPERATURA_C  NUMBER(5,2));
ALTER TABLE SENSORES MODIFY (N NUMBER(10,0));
ALTER TABLE SENSORES MODIFY (P NUMBER(10,0));
ALTER TABLE SENSORES MODIFY (K NUMBER(10,0));
```

---

## 🧠 Dificuldades e Soluções

| Dificuldade                                            | Causa                                             | Solução                                            |
| ------------------------------------------------------ | ------------------------------------------------- | -------------------------------------------------- |
| **Perda de casas decimais (4.63 → 463)**               | Configuração incorreta do separador decimal       | `ALTER SESSION SET NLS_NUMERIC_CHARACTERS = '.,';` |
| **ORA-00957: nome de coluna duplicado**                | Criação automática duplicou a coluna `PH`         | Criação manual da tabela                           |
| **ORA-01438: valor maior que a precisão especificada** | Tipos `NUMBER(3,1)` e `NUMBER(2,1)` insuficientes | Alteração para `NUMBER(4,2)`                       |
| **Tipos incorretos ao importar**                       | Wizard do Oracle gera `NUMBER(38,0)` por padrão   | Ajuste manual dos tipos antes da importação        |

---

## 📥 Etapas do Processo

1. **Configuração inicial**

   ```sql
   ALTER SESSION SET NLS_NUMERIC_CHARACTERS = '.,';
   ```
2. **Criação da tabela com tipos ajustados.**
3. **Importação da planilha Excel:**

   * Método: *Insert into existing table*
   * Verificação dos tipos e escala numérica.
4. **Validação pós-importação:**

   ```sql
   SELECT * FROM SENSORES FETCH FIRST 10 ROWS ONLY;
   ```

---

## 🔍 Consultas SQL Funcionais

### 1️⃣ Leituras gerais

```sql
SELECT * FROM SENSORES FETCH FIRST 20 ROWS ONLY;
```

### 2️⃣ Leituras com pH fora da faixa ideal (5,5–6,8)

```sql
SELECT N, P, K, PH, AJUSTE_PH, MENSAGEM
FROM SENSORES
WHERE PH < 5.5 OR PH > 6.8;
```

### 3️⃣ Situações de emergência (≥40°C e umidade <55%)

```sql
SELECT N, P, K, PH, UMIDADE, TEMPERATURA_C, MENSAGEM
FROM SENSORES
WHERE TEMPERATURA_C >= 40 AND UMIDADE < 55;
```

### 4️⃣ Médias gerais

```sql
SELECT
  ROUND(AVG(PH),2) AS PH_MEDIO,
  ROUND(AVG(UMIDADE),2) AS UMIDADEMEDIA,
  ROUND(AVG(TEMPERATURA_C),2) AS TEMP_MEDIA
FROM SENSORES;
```

### 5️⃣ Diagnóstico automatizado

```sql
SELECT
  N, P, K, PH, AJUSTE_PH, UMIDADE, TEMPERATURA_C,
  CASE
    WHEN TEMPERATURA_C >= 40 AND UMIDADE < 55 THEN 'EMERGÊNCIA: calor extremo (>=40°C e umidade <55%)'
    WHEN PH < 5.5 OR PH > 6.8 THEN 'ALERTA: pH fora da faixa ideal (5,5–6,8)'
    ELSE 'CONDIÇÃO NORMAL'
  END AS DIAGNOSTICO
FROM SENSORES;
```

---

## 📈 Resultados Obtidos

| N | P | K | PH   | AJUSTE_PH | UMIDADE_ | TEMPERATURA_C | MENSAGEM                                                     |
| - | - | - | ---- | --------- | -------- | ------------- | ------------------------------------------------------------ |
| 0 | 0 | 0 | 4.63 | 0.60      | 43.00    | 28.80         | Bomba=OFF → pH fora (alvo 5.5–6.8)                           |
| 0 | 0 | 0 | 6.53 | 1.60      | 39.50    | 42.60         | Bomba=ON → EMERGÊNCIA: calor extremo (>=40°C e umidade <55%) |

✅ Dados importados corretamente
✅ Precisão decimal preservada
✅ Consultas executadas com sucesso no Oracle SQL Developer

---

## 💡 Conclusão

O processo comprovou a importância de:

* Configurar corretamente a sessão (`NLS_NUMERIC_CHARACTERS`);
* Ajustar a precisão de campos decimais no Oracle;
* Tratar erros de importação e validar a integridade pós-carga.

Os dados gerados no **VS Code (Fase 2 – Capítulo 1 do projeto Farm Tech Solutions)** foram corretamente transformados e analisados dentro do **Oracle Database**, refletindo com exatidão as medições obtidas via **simulador Wokwi**.

---

## 🗃 Histórico de Versões

| Versão    | Data       | Descrição                                                                                         |
| --------- | ---------- | ------------------------------------------------------------------------------------------------- |
| **1.0.0** | 12/11/2025 | Inclusão da origem dos dados (VS Code/Wokwi – Fase 2 Cap. 1 Farm Tech Solutions) e revisão final. |
| **0.9.0** | 11/11/2025 | Correção de erros ORA e adequação de tipos numéricos.                                             |
| **0.8.0** | 10/11/2025 | Importação inicial da planilha e testes de precisão.                                              |
| **0.7.0** | 09/11/2025 | Criação da estrutura da tabela e ambiente Oracle.                                                 |

---

## 📋 Licença

Projeto desenvolvido para fins **acadêmicos** – **FIAP**.
Reprodução ou redistribuição restrita ao contexto educacional.

---

```

---

Deseja que eu gere este conteúdo como **arquivo `README.md` pronto para download** para você subir diretamente no GitHub?
```
