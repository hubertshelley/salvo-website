---
title: "Serve Static"
weight: 8060
menu:
  book:
    parent: "middlewares"
---

將靜態文件或者內嵌的文件作為服務提供的中間件.

## 配置 Cargo.toml

```toml
salvo = { version = "*", features = ["serve-static"] }
```

## 主要功能

* `StaticDir` 提供了對靜態本地文件夾的支持. 可以將多個文件夾的列表作為參數. 比如:

    ```rust
    use salvo_core::routing::Router;
    use salvo_core::Server;
    use salvo_extra::serve::StaticDir;

    #[tokio::main]
    async fn main() {
        tracing_subscriber::fmt().init();
        
        let router = Router::new()
            .path("<**path>")
            .get(StaticDir::new(vec!["examples/static/body", "examples/static/girl"]));
        Server::new(TcpListener::bind("127.0.0.1:7878")).serve(router).await;
    }
    ```
    如果在第一個文件夾中找不到對應的文件, 則會到第二個文件夾中找.

* 提供了對 `rust-embed` 的支持, 比如:
    ```rust
    use rust_embed::RustEmbed;
    use salvo::prelude::*;
    use salvo::serve_static::static_embed;

    #[derive(RustEmbed)]
    #[folder = "static"]
    struct Assets;

    #[tokio::main]
    async fn main() {
        tracing_subscriber::fmt().init();

        let router = Router::with_path("<**path>").get(static_embed::<Assets>().with_fallback("index.html"));
        tracing::info!("Listening on http://127.0.0.1:7878");
        Server::new(TcpListener::bind("127.0.0.1:7878")).serve(router).await;
    }
    ```

    `with_fallback` 可以設置在文件找不到時, 用這裏設置的文件代替, 這個對應某些單頁網站應用來還是有用的.