# Projeto Integrador Avançado - AeroMotion Dynamics

Pacote de análise industrial com notebook, relatório técnico, modelos, API e pipeline OpenCV.

## Entregáveis

- `analises/analise.ipynb`: inspeção, saneamento features temporais, EDA e avaliação dos modelos, notebook executado.
- `relatorio.pdf`: relatório técnico da Etapa 1.
- `parecer.pdf`: parecer de quatro páginas da Etapa 3.
- `artifacts/model.joblib`, `artifacts/isolation_forest.joblib`, `artifacts/metrics.json`: modelos exportados e métricas.
- `src/pipeline.py`: tratamento e treino temporal.
- `src/vision.py`: inspeção geométrica por Canny e segmentação térmica HSV.
- `src/api.py`: FastAPI com `/predict`, `/predict-batch`, `/health` e `/docs`.
- `src/static/index.html`: painel web responsivo servido em `/`, com análise integrada à API.
- `Dockerfile`, `docker-compose.yml`, `requirements-api.txt`: implantação em container.
- `planos/mlops.md`: plano de monitoramento e governança.

## Reproduzir

Requer Python 3.12, instale as dependências a partir da raiz do projeto:

```bash
python -m pip install -r requirements-dev.txt
python -m src.pipeline dados/sensores.csv --out artifacts
python analises/gera.py
python scripts/rel.py
python -m pytest -q
python -m uvicorn src.api:app --host 0.0.0.0 --port 8000
```

Abra `http://127.0.0.1:8000/` para usar o painel. A documentação interativa da API está em `http://127.0.0.1:8000/docs`.

Para iniciar o container, após gerar os artefatos use `docker compose up --build`.

## Limites de validação

O modelo usa um corte cronológico único do CSV fornecido: 70% treino e 30% teste. O melhor modelo foi selecionado apenas por validação temporal interna ao treino. O recall por classe de falha no bloco final ficou acima de 90%, próximo do limiar para desgaste de rolamento. Isso não comprova generalização para produção. O CSV não contém imagens, então o código OpenCV foi verificado com imagens sintéticas e ainda requer calibração e dados industriais rotulados. A API mantém janelas móveis de 1 h e 6 h em memória por processo, rejeita leituras fora de ordem e não persiste o histórico após reinício; produção deve compartilhar histórico entre réplicas.
