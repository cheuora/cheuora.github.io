---
layout: post
title: gradle로 Springboot 시작하기
tags: [gradle, SpringBoot]
---

이 글은 이클립스에서의 설정이 아닌 터미널에서 gradle로 스프링부트를 설치하는 방법을 적어놓은 것이다.

실행 환경은 아래와 같다. 

* Ubuntu 20.04 LTS
* openjdk version "1.8.0_312"

### gradle 설치

이 글을 쓰고 있는 현재 gradle의 최신 버전은 7.4.2이다. 최신 버전 확인은 

https://gradle.org/releases/

에서 확인하기 바란다. 

아쉽게도 ubuntu에서 `apt install gradle`을 실행하면 옛날 4.x대의 버전이 설치가 된다. 최신 버전은 직접 다운받아야 한다. 

```bash
$ VERSION=7.4.2
$ wget https://services.gradle.org/distributions/gradle-${VERSION}-bin.zip -P /tmp
```

압축을 푼다.

```bash
$ sudo unzip -d /opt/gradle /tmp/gradle-${VERSION}-bin.zip
```

링크를 연결한다.

```
$ sudo ln -s /opt/gradle/gradle-${VERSION} /opt/gradle/latest
```



터미널에서 gradle명령을 가능하게 하려면 환경설정을 해야 한다.

아래 파일을 열어서(아마 없을 것이다. 새로 만들면 된다)

```
$ sudo vi /etc/profile.d/gradle.sh
```

다음과 같이 입력하고 저장한다.

```
export GRADLE_HOME=/opt/gradle/latest
export PATH=${GRADLE_HOME}/bin:${PATH}
```

스크립트 파일에 실행 권한을 부여한다. 

```
$ sudo chmod +x /etc/profile.d/gradle.sh
```

source 명령어로 스크립트 로딩

```
$ source /etc/profile.d/gradle.sh
```



터미널에서 아래와 같이 버전확인 가능하면 OK

```
$ gradle -version

------------------------------------------------------------
Gradle 7.4.2
------------------------------------------------------------
```



이제 `gradle init` 명령으로 프로젝트를 하나 생성한다. 

```
~/gradletest$ gradle init
Starting a Gradle Daemon (subsequent builds will be faster)

Select type of project to generate:
  1: basic
  2: application
  3: library
  4: Gradle plugin
Enter selection (default: basic) [1..4]  <-- 1선택

Select build script DSL:
  1: Groovy
  2: Kotlin
Enter selection (default: Groovy) [1..2] <-- 1선택

Generate build using new APIs and behavior (some features may change in the next minor release)? (default: no) [yes, no]  <-- no선택

Project name (default: gradletest): <-- default선택
```



생성된 프로젝트의 폴더 구조는 다음과 같다. 

```
~/gradletest$ tree .
.
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
└── settings.gradle
```



### SpringBoot반영

gradle의 반영은 build.gradle파일에서 해 준다. 

```gradle
/*
 * This file was generated by the Gradle 'init' task.
 *
 * This is a general purpose Gradle build.
 * Learn more about Gradle by exploring our samples at https://docs.gradle.org/7.4.2/samples
 */

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.5.10.RELEASE")
    }
}

plugins {
    id 'org.springframework.boot' version '2.5.0'
}
apply plugin: 'java'
apply plugin: 'idea'
apply plugin: 'io.spring.dependency-management'


jar {
    baseName = 'gs-spring-boot'
    version =  '0.1.0'
}

repositories {
    mavenCentral()
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    testImplementation("junit:junit")
}
```



#### Controller.java생성

다음과 같이 controller.java파일을 생성해 보자

```
$ mkdir -p src/main/java/com/runTestSample
$ cd src/main/java/com/runTestSample
$ touch Controller.java
```



```
//file: Controller.java 
package com.runTestSample;

import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;

@RestController
public class Controller {

    @RequestMapping("/")
    public String index() {
        return "<h1>와 이게 왜 되지</h1>";
    }

    @RequestMapping("/test")
    public String test() {
        return "<h1>여기는 테스트.</h1>";
    }
}
```



#### Application.java 생성

동일한 디렉토리(`src/main/java/com/runTestSample`)에 아래와 같이 생성한다.

```
package com.runTestSample;

import java.util.Arrays;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    public CommandLineRunner commandLineRunner(ApplicationContext ctx) {
        return args -> {

            System.out.println("Let's inspect the beans provided by Spring Boot:");

            String[] beanNames = ctx.getBeanDefinitionNames();
            Arrays.sort(beanNames);
            for (String beanName : beanNames) {
                System.out.println(beanName);
            }

        };
    }
}
```



### 확인

이제 잘 동작하는지 확인해 보자

```
$ gradle build

BUILD SUCCESSFUL in 1s
4 actionable tasks: 4 executed
```



실행을 위한 jar파일은 `build/libs` 밑에 생성된다. 

```
~/gradletest$ tree build
build
├── bootJarMainClassName
├── classes
│   └── java
│       └── main
│           └── com
│               └── runTestSample
│                   ├── Application.class
│                   └── Controller.class
├── generated
│   └── sources
│       ├── annotationProcessor
│       │   └── java
│       │       └── main
│       └── headers
│           └── java
│               └── main
├── libs
│   ├── gradletest.jar
│   └── gs-spring-boot-0.1.0-plain.jar
└── tmp
    ├── bootJar
    │   └── MANIFEST.MF
    ├── compileJava
    │   └── previous-compilation-data.bin
    └── jar
        └── MANIFEST.MF

```



SpringBoot구동을 해 보자. 

```
$ java -jar build/libs/gradletest.jar
```

웹 브라우저를 띄우고 http://localhost:8080으로 접속

![](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2022/20220412_1.png)



http://localhost:8080/test로도 접속하면 다음과 같이 뜬다. 

![](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2022/20220412_2.png)

