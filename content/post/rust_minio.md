---
title: "Rust操作MinIO"
date: 2024-01-26T10:57:48+08:00
tags: ["Rust", "minio-rs", "minio"]
categories: ["Rust"]
keywords: ["minio-rs", "Rust操作minio"]
draft: true
---

在项目中经常会碰到一些小文件需要保存，而大量的小文件会导致磁盘的inode被占用，从而导致磁盘空间不足。MinIO是一个开源的对象存储服务器，用来存储和管理大量的非结构化数据。这篇文章主要整理Rust 对MinIO简单操作的代码例子。

## 启动Docker环境

```bash
docker run \
   -p 9000:9000 \
   -p 9001:9001 \
   --name minio \
   -e "MINIO_ROOT_USER=fantastic" \
   -e "MINIO_ROOT_PASSWORD=fantastic" \
   quay.io/minio/minio server /data --console-address ":9001"
```

## 代码例子

添加相关依赖：

```toml
minio = "0.1.0"
tokio = {version = "1.4", features = ["full"]}
uuid = { version ="1.7",features = ["v4", "fast-rng", "macro-diagnostics"]}
```

完整的代码例子，包含了上传文件和读取文件

```rust
use std::path::Path;

use minio::s3::args::{BucketExistsArgs, GetObjectArgs, MakeBucketArgs, UploadObjectArgs};
use minio::s3::client::Client;
use minio::s3::creds::StaticProvider;
use minio::s3::http::BaseUrl;
use minio::s3::response::UploadObjectResponse;
use uuid::Uuid;

const BASEURL: &str = "http://127.0.0.1:9000";
const ACCESSKEY: &str = "sEmZO3o3UqEdYIvyVipD";
const SECRETACCESS: &str = "Fc7pbGbcEIBdjveMbdyBjVWmgmOUMbi9TSuccjSA";
const BUCKETNAME: &'static str= "demo";

pub type Error = Box<dyn std::error::Error + Send + Sync>;
pub type Result<T> = std::result::Result<T, Error>;


pub struct MinioConfig {
    base_url: String,
    access_key: String,
    secret_access: String,
}

impl MinioConfig{
    pub fn new(base_url: String, access_key: String,secret_access:String) -> MinioConfig {
        return MinioConfig{
            base_url,
            access_key,
            secret_access,
        };
    }
}

pub struct MinioClient {
    client:Client,
}

impl MinioClient{
    pub fn new(minio_config: &MinioConfig) -> Result<MinioClient> {
        let static_provider = StaticProvider::new(
            &minio_config.access_key,
            &minio_config.secret_access,
            None,
        );
        let base_url = minio_config.base_url.parse::<BaseUrl>()?;
        let client = Client::new(
            base_url,
            Some(Box::new(static_provider)),
            None,
            None,
        )?;
        Ok(MinioClient{client})
    }

    pub async fn init_bucket(&self)-> Result<()>{
        let exists = self.client
            .bucket_exists(&BucketExistsArgs::new(BUCKETNAME)?)
            .await?;
        // Make 'asiatrip' bucket if not exist.
        if !exists {
            self.client
                .make_bucket(&MakeBucketArgs::new(BUCKETNAME)?)
                .await?;
        }
        Ok(())
    }

    pub async fn upload_object(&self, full_filename: &str) ->Result<UploadObjectResponse> {
        let filename = match Path::new(full_filename).file_name() {
            Some(name) => name.to_str().ok_or_else(|| "Invalid UTF-8")?,
            None => return Err("Invalid file path".into()),
        };
        let uuid_str = Uuid::new_v4().to_string();
        let object_name = format!("{}-{}",uuid_str,filename);
        let upload_object_response = self.client
            .upload_object(
                &UploadObjectArgs::new(
                    BUCKETNAME,
                    &object_name,
                    &full_filename,
                )?
            )
            .await?;
        Ok(upload_object_response)
    }

    pub async fn get_object(&self, object_name: &str) -> Result<()>{
        let get_object_response = self.client.get_object(
            &GetObjectArgs::new(BUCKETNAME, object_name)?
        )
        .await?;
        let content = get_object_response.text().await?;
        println!("{}",content);
        Ok(())
    }
}

#[tokio::main]
async fn main() ->  Result<()>  {
    let minio_config = MinioConfig::new(
        BASEURL.to_string(),
        ACCESSKEY.to_string(),
        SECRETACCESS.to_string(),
    );
    let client = MinioClient::new(&minio_config)?;
    client.init_bucket().await?;

    let upload_object_response = client.upload_object("/dev/about_minio/minio_demo/Cargo.toml").await?;
    println!("filename is {}", upload_object_response.object_name);
    client.get_object(&upload_object_response.object_name).await?;
    Ok(())

}
```

## 相关链接

- <https://min.io/docs/minio/container/index.html>
- <https://github.com/minio/minio-rs>
