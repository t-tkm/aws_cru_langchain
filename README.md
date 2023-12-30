# aws_cru_langchain
This project uses LLM and LangChain to implement natural language SQL for AWS Cost and Usage Reports (CUR).

## 参考サイト
コードは、こちらのサイトがベース(本家)になります。
- [Introducing the LangChain integration](https://cube.dev/blog/introducing-the-langchain-integration)
- https://github.com/cube-js/cube/tree/master/examples/langchain

## System
![system](https://raw.githubusercontent.com/t-tkm/blog_images/main/2023/aws_cost_usage_report3/img1.png)

## Getting Started
### Cube Core起動
「[AWS Cost Usage Reportの可視化(2) -ヘッドレスBIツールCubeを試してみる](https://t-tkm.github.io/blog/posts/2023/09/aws_cost_usage_report2/)」のStep1を参考にする。

- cure core起動パラメータ準備
    ```zsh
    touch .env
    ```
    .envファイル
    ```file
    CUBEJS_AWS_KEY=<AWSアクセスキー>
    CUBEJS_AWS_SECRET=<AWSシークレットキー>
    CUBEJS_AWS_REGION=ap-northeast-1
    CUBEJS_AWS_S3_OUTPUT_LOCATION=s3://<S3バケット名>
    CUBEJS_DB_TYPE=athena
    CUBEJS_EXTERNAL_DEFAULT=true
    CUBEJS_SCHEDULED_REFRESH_DEFAULT=true
    CUBEJS_DEV_MODE=true
    CUBEJS_SCHEMA_PATH=model
    ```

- cube coreコンテナ起動
    ```zsh
    docker compose up -d
    ```

- ブラウザでcubeへ接続し(http://localhost:4000/)、(Athenaの)モデル作成。

### LangChainアプリ起動
- python環境を準備
    ```zsh
    (base)$ conda create -n aws_cur_langchain phthon=3.1
    (base)$ conda activate aws_cur_langchain
    (aws_cur_langchain)$ pip install -r requirements.txt
    ```
    ※ventやvirtualenvなど、お好みのツールでok。上記は、conda(Anacondaディストリビューション)を使う場合の参考。

- OpenAI APIのキー取得(ex. sk-XXXXXXXXXXX)

- Cube APIシークレットキー(ex. JWTフォーマット)。ブラウザでCube Coreへアクセスし、「Playground」->「Code」から取得できる。

    ```
    (snip)
    const cubejsApi = cubejs(
    '{ヘッダ}.{ペイロード}.{署名}',
    { apiUrl: 'http://localhost:4000/cubejs-api/v1' }
    );
    (snip)
    ```  


- .envファイルに、下記の環境変数を追加。
    .envファイル
    ```file
    OPENAI_API_KEY=<OpenAI APIのキー>
    CUBE_API_URL=http://localhost:4000/cubejs-api/v1
    CUBE_API_SECRET=<Cube APIシークレットキー>
    #DATABASE_URL=postgresql://localhost:4000/test
    DATABASE_URL=host=localhost port=15432 dbname=test user=username password=password
    ```

- アプリ起動
    ```zsh
    (aws_cur_langchain)$ streamlit run streamlit_app.py
    ``` 
    ※初回起動時はIngestion処理のため(「Loading context from Cube API...」)5分程度待つ。

- GUIが表示されたらクエリ投入(ex. pricing_public_on_demand_costの合計は？)
