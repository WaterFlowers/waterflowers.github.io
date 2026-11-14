---
title: "Server configuration test"
date: 2026-02-20
draft: false
tags: ["Data", "Server", "test"]
summary: "Server configuration test"
---
## Server configuration test
---

### 1. 准备依赖

在你的Ubuntu虚拟机里，确保已经安装了必要的库：  
sudo apt install build-essential cmake libssl-dev -y  
然后创建项目文件夹，下载两个头文件库（只需头文件，无需编译）：  
mkdir ai_server && cd ai_server  
wget https://raw.githubusercontent.com/yhirose/cpp-httplib/master/httplib.h  
wget https://github.com/nlohmann/json/releases/latest/download/json.hpp  

### 2. 编写服务器主程序 main.cpp

```cpp
#include "httplib.h"
#include "json.hpp"
#include <iostream>

using json = nlohmann::json;

int main() {
    httplib::Server svr;

    // 接收UE5发来的POST请求
    svr.Post("/chat", [](const httplib::Request& req, httplib::Response& res) {
        try {
            // 1. 解析UE5发来的JSON，提取文本
            auto req_json = json::parse(req.body);
            std::string user_text = req_json["text"];

            std::cout << "收到UE5文本: " << user_text << std::endl;

            // 2. 向DeepSeek API发送请求
            httplib::Client deepseek("https://api.deepseek.com");
            deepseek.set_default_headers({
                {"Authorization", "Bearer My_DEEPSEEK_API_KEY"},
                {"Content-Type", "application/json"}
            });

            json ds_body = {
                {"model", "deepseek-chat"},
                {"messages", json::array({
                    {{"role", "user", "content", user_text}}
                })},
                {"stream", false}
            };

            auto ds_res = deepseek.Post("/chat/completions", ds_body.dump(), "application/json");

            // 3. 解析DeepSeek的返回，提取回答文本
            if (ds_res && ds_res->status == 200) {
                auto ds_json = json::parse(ds_res->body);
                std::string reply = ds_json["choices"][0]["message"]["content"];
                
                json resp;
                resp["reply"] = reply;
                resp["status"] = "ok";
                res.set_content(resp.dump(), "application/json");
            } else {
                json err;
                err["status"] = "error";
                err["message"] = "DeepSeek API调用失败";
                res.status = 500;
                res.set_content(err.dump(), "application/json");
            }
        } catch (const std::exception& e) {
            json err;
            err["status"] = "error";
            err["message"] = std::string("服务器内部错误: ") + e.what();
            res.status = 500;
            res.set_content(err.dump(), "application/json");
        }
    });

    std::cout << "服务器启动: http://0.0.0.0:8080" << std::endl;
    svr.listen("0.0.0.0", 8080);
    return 0;
}

```

### 3. 编写 CMakeLists.txt
cmake_minimum_required(VERSION 3.10)  
project(AIServer)  
set(CMAKE_CXX_STANDARD 17)  
find_package(OpenSSL REQUIRED)  
add_executable(ai_server main.cpp)  
target_link_libraries(ai_server PRIVATE pthread OpenSSL::SSL OpenSSL::Crypto)  

### 4. 编译并运行

mkdir build && cd build  
cmake ..  
make  
./ai_server  

终端显示“服务器启动: http://0.0.0.0:8080”就说明成功了。  

### 5. 测试服务器
可以用 curl 模拟UE5客户端：  
curl -X POST http://localhost:8080/chat \  
  -H "Content-Type: application/json" \  
  -d '{"text":"你好，介绍一下长安城的漕运系统"}'  
如果返回了AI回答的JSON，说明链路完全打通。  
 

 