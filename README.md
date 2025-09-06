# 🚀 Spark + Delta Lake com VS Code Remote

Este setup fornece um **container Docker enxuto** para trabalhar com **Apache Spark 3.5.3** e **Delta Lake**, permitindo desenvolvimento remoto com **VS Code Remote - Containers**.  
É ideal para estudos e também pode servir como base para ambientes Spark mais complexos, sem a necessidade de configurar Spark, Java ou Python manualmente.

O volume mapeado garante que notebooks e dados persistam após reiniciar o container, e configurações adicionais do Spark (como memória ou número de cores) podem ser ajustadas via variáveis de ambiente no `docker run`.

A imagem final possui aproximadamente **2.7 GB**, o que é considerado normal para ambientes Spark, já que inclui Spark, Hadoop, Java, Scala e dependências adicionais do Python. Esse tamanho está dentro da média (2–4 GB) encontrada em setups semelhantes. Para quem busca algo mais leve, seria necessário montar uma imagem baseada em versões “slim” do Python ou do OpenJDK e instalar o Spark manualmente, mas isso aumenta a complexidade do setup.

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

USER root

RUN apt-get update && \
    apt-get install -y --no-install-recommends git && \
    pip install --no-cache-dir delta-spark==3.3.2 ipykernel && \
    rm -rf /var/lib/apt/lists/*
```

**O que ele faz agora, detalhado:**

1. **Imagem base:** Utiliza `bitnami/spark:3.5.3`, que já inclui Spark, Java, Hadoop e Scala.
2. **Diretório de trabalho:** Define `/home` como diretório de trabalho dentro do container.
3. **Usuário root:** Permite instalar pacotes do sistema.
4. **Instalações adicionais:**

   * `git`: para versionamento e clonagem de repositórios.
   * `delta-spark==3.3.2`: para integração com Delta Lake no Spark.
   * `ipykernel`: para executar notebooks Python dentro do container.
5. **Limpeza:** Remove cache do `apt` para reduzir o tamanho final da imagem.

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

## ⚠️ Compatibilidade de versões

Um detalhe muitas vezes negligenciado, mas que pode causar **erros difíceis de diagnosticar**, é a compatibilidade entre as ferramentas.
No ecossistema Spark + Delta Lake, as versões precisam estar alinhadas para que tudo funcione corretamente.

No exemplo deste setup, usamos:

* **Apache Spark:** 3.5.3
* **Delta Lake (delta-spark):** 3.3.2

Essa escolha não é aleatória: a compatibilidade entre **Spark 3.5.x** e **Delta Lake 3.3.x** é documentada oficialmente.
Negligenciar esse cuidado pode gerar problemas como falhas ao inicializar sessões Spark, erros de schema ou até corromper tabelas Delta.

### 🔗 Compatibilidade oficial (Delta Lake x Spark)

| Delta Lake version | Apache Spark version  |
| ------------------ | --------------------- |
| 4.0.x              | 4.0.x                 |
| 3.3.x              | 3.5.x                 |
| 3.2.x              | 3.5.x                 |
| 3.1.x              | 3.5.x                 |
| 3.0.x              | 3.5.x                 |
| 2.4.x              | 3.4.x                 |
| 2.3.x              | 3.3.x                 |
| 2.2.x              | 3.3.x                 |
| 2.1.x              | 3.3.x                 |
| 2.0.x              | 3.2.x                 |
| 1.2.x              | 3.2.x                 |
| 1.1.x              | 3.2.x                 |
| 1.0.x              | 3.1.x                 |
| 0.7.x / 0.8.x      | 3.0.x                 |
| < 0.7.0            | 2.4.2 – 2.4.\<latest> |

📖 Fonte: [Documentação oficial do Delta Lake](https://docs.delta.io/releases/#compatibility-with-apache-spark)

---

👉 **Case prático:**
Quando montei o setup inicial, ignorei esse detalhe de compatibilidade e perdi horas tentando corrigir erros que não faziam sentido.
Só depois de voltar à documentação oficial percebi que estava usando uma versão do Delta incompatível com o Spark — o que tornava o ambiente instável.

Esse episódio reforçou ainda mais o aprendizado: **não existe atalho fora da documentação oficial**. Conferir tabelas de compatibilidade como essa deve ser sempre o primeiro passo antes de configurar qualquer ambiente.

---

## 💡 Aprendizado

O grande aprendizado que fica é simples, mas essencial: **sempre focar na documentação oficial**.

Negligenciar esse passo é uma falha grave — foi exatamente isso que aconteceu comigo.
Os melhores sempre repetem essa lição, mas eu ignorei e acabei pagando o preço.

O caminho mais rápido e seguro nunca é sair tentando milhares de soluções de vídeos, fóruns ou jogando perguntas no GPT sem critério.
O verdadeiro atalho é **ler e entender a documentação oficial, direto de quem desenvolveu**.

Quando não seguimos isso, caímos no imediatismo: testamos mil coisas sem compreender de fato, gastamos tempo demais procurando respostas rápidas e, no fim, a solução estava lá desde o início na documentação.
