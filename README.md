# Análise de Dados de Demissões Globais

## Descrição

Este projeto envolveu a realização de limpeza de dados e análise exploratória em um conjunto de dados abrangente sobre Demissões Globais, utilizando MySQL. O objetivo principal foi transformar dados brutos em um formato estruturado e informativo, por meio de consultas SQL avançadas para obter insights importantes.
[Fonte de dados](https://www.kaggle.com/datasets/happyude/world-layoffs)

## Processo de Limpeza de Dados

### 1. Remover Valores Duplicados

- **Criar tabela staging:**
  ```sql
  CREATE TABLE layoffs_staging LIKE layoffs;

  INSERT INTO layoffs_staging
  SELECT * FROM layoffs;
  ```

- **Identificar e remover linhas duplicadas:**
  ```sql
  WITH duplicate_cte AS (
      SELECT *,
      ROW_NUMBER() OVER (
          PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
      ) AS row_num
      FROM layoffs_staging
  )
  SELECT * FROM duplicate_cte WHERE row_num > 1;

  DELETE FROM layoffs_staging2 WHERE row_num > 1;
  ```

### 2. Padronizar os Dados

- **Remover espaços em branco dos nomes das empresas:**
  ```sql
  UPDATE layoffs_staging2
  SET company = TRIM(company);
  ```

- **Padronizar os nomes dos setores industriais:**
  ```sql
  UPDATE layoffs_staging2
  SET industry = 'Crypto'
  WHERE industry LIKE 'Crypto%';
  ```

- **Remover pontos finais dos nomes dos países:**
  ```sql
  UPDATE layoffs_staging2
  SET country = TRIM(TRAILING '.' FROM country)
  WHERE country LIKE 'United States%';
  ```

- **Alterar o tipo de dado da coluna `data`:**
  ```sql
  UPDATE layoffs_staging2
  SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');

  ALTER TABLE layoffs_staging2
  MODIFY COLUMN `date` DATE;
  ```

### 3. Lidar com Valores Nulos

- **Deletar linhas onde `total_laid_off` e `percentage_laid_off` são NULL:**
  ```sql
  DELETE FROM layoffs_staging2
  WHERE total_laid_off IS NULL
  AND percentage_laid_off IS NULL;
  ```

- **Imputar dados de setor industrial ausentes:**
  ```sql
  UPDATE layoffs_staging2 t1
  JOIN layoffs_staging2 t2
  ON t1.company = t2.company
  SET t1.industry = t2.industry
  WHERE t1.industry IS NULL
  AND t2.industry IS NOT NULL;
  ```

### 4. Remover Colunas Desnecessárias

- **Excluir coluna `row_num`:**
  ```sql
  ALTER TABLE layoffs_staging2
  DROP COLUMN row_num;
  ```

## Dados Finais Limpos

Os dados limpos são armazenados na tabela `layoffs_staging2`, que agora contém registros padronizados e sem duplicatas, com os valores ausentes tratados de forma adequada.

## Insights sobre os Dados
Alguns insights gráficos podem ser vistos no arquivo `visualizacao insights.ipynb`. São eles:
1. Evolução das demissões ao longo do tempo
2. Setores mais afetados
3. Demissões por localização (Países mais afetados)
4. Distribuição percentual de demissões nas empresas

## Como Executar o Projeto

1. **Clonar o repositório:**
    ```bash
    git clone https://github.com/ramoncampos/analise_dados_demissoes_globais.git
    
    ```

2. **Configurar o banco de dados e importar os dados:**
    - Importe o arquivo `demissoes.csv` para sua base de dados MySQL.
    - Execute o scrit SQL `demissoes_globais_data_cleaning.sql` para criar as tabelas necessárias e limpar os dados

3. **Run the analysis:**
    Execute o script SQL `demissoes_globais_eda.sql` para gerar insights a partir dos dados limpos.

## Conclusão

Ao limpar e padronizar completamente o conjunto de dados World Layoffs, este projeto demonstra a eficácia do SQL na preparação de dados para uma análise significativa. Os insights resultantes podem fornecer informações valiosas sobre tendências e padrões de demissões em nível global.
