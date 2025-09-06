# 🚀 Spark + Delta Lake com VS Code Remote

Este setup fornece um **container Docker enxuto** para trabalhar com **Apache Spark 3.5.3** e **Delta Lake**, permitindo desenvolvimento remoto com **VS Code Remote - Containers**.

É ideal para estudos e serve como base para setups, sem precisar configurar Spark, Java ou Python manualmente.

---

## 📦 Configuração do container

* **Base:** `bitnami/spark:3.5.3` (Debian 12, Spark 3.5.3, OpenJDK 17)
* **Python:** 3.12.8
* **PySpark:** 3.5.3
* **Scala:** 2.12.18
* **Delta Lake:** delta-spark 3.3.2
* **Diretório de trabalho:** `/home`
* **Dependências adicionais:** `ipykernel` (para notebooks)

---

## 🐳 Dockerfile

```dockerfile
FROM bitnami/spark:3.5.3

WORKDIR /home

RUN pip install --no-cache-dir \
    delta-spark==3.3.2 \
    ipykernel
```

**O que ele faz:**

1. Baseia-se na imagem oficial do Spark com Java, Hadoop e Scala já instalados.
2. Define `/home` como diretório de trabalho.
3. Instala `delta-spark` e `ipykernel` para suportar Delta Lake e notebooks Python.

---

## ⚙️ Passos para rodar o container

1. Crie o diretório local que será usado como volume:

```bash
mkdir spark-delta-project
cd spark-delta-project
```



2. Construa a imagem Docker:

```bash
docker build -t spark-delta-jupyter .
```

3. Rode o container:

```bash
docker run -d \
  -u 0 \
  -v $(pwd):/home \
  --name spark-delta-container \
  spark-delta-jupyter
```

**Explicação dos parâmetros:**

* `-d` → roda o container em **modo detach** (em segundo plano).
* `-u 0` → executa como **root** dentro do container (evita problemas de permissão).
* `-v $(pwd):/home` → mapeia o diretório local para `/home` do container.
* `--name spark-delta-container` → nome do container, facilitando operações futuras.
* `spark-delta-jupyter` → nome da imagem construída.

---

## 🧪 Testando Delta Lake

Crie um script Python ou notebook:

```python
from pyspark.sql import SparkSession
from delta import configure_spark_with_delta_pip

spark = configure_spark_with_delta_pip(
    SparkSession.builder
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension")
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog")
).getOrCreate()

df = spark.createDataFrame([("Alice", 34), ("Bob", 45)], ["name", "age"])
df.write.format("delta").mode("overwrite").save("/tmp/delta-table")
spark.read.format("delta").load("/tmp/delta-table").show()
```

> ✅ Isso verifica se o **Delta Lake** está funcionando corretamente no container.

---

## 🖥 VS Code Remote Connection

O **Remote - Containers** permite trabalhar **diretamente dentro do container**:

### Benefícios

* Evita instalar Spark, Java ou Python localmente.
* Ambiente isolado e reprodutível.
* Acesso direto a notebooks e scripts com Spark + Delta Lake prontos.
* Facilita setups complexos sem configuração manual.

### Como usar

1. Instale a extensão [Remote - Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers).
2. Abra o VS Code no diretório local do projeto.
3. Pressione `F1` → **Remote-Containers: Attach to Running Container…**
4. Escolha `spark-delta-container`.
5. O VS Code estará conectado ao container; execute notebooks, scripts ou terminais Python com Spark e Delta prontos.

---

## ⚠️ Observações

* Este setup é **inicial e enxuto**, servindo como base para ambientes Spark.
* O volume mapeado garante que notebooks e dados persistam após reiniciar o container.
* Configurações adicionais do Spark (memória, cores, etc.) podem ser ajustadas via variáveis de ambiente no `docker run`.
