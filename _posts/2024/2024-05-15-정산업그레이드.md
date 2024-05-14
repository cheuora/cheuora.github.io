---
layout: post
title: 정산내역 확인 발송 시스템 리뉴얼
tags: [unoserver,chokidar]
use_math: false
---

## 정산내역 확인 발송 시스템 업그레이드

기존 버전의 정산내역 확인 발송 시스템의 문제가 있었다. 

1. 가끔 정산 파일의 Sending의 누락 
2. 프로세스의 다운



정산파일은 pdf변환시 걸리는 시간 및 엑셀에 이미지 삽입 작업들의 동기화 문제 때문에 가끔 정산 파일 작업이 누락되는 경우가 있었다. 그리고 오래동안 프로세스를 띄워놓고 있다 보면 가끔 프로세스가 죽는 경우도 있었다. 

다음과 같은 방향으로 개선을 해야 겠다고 생각했다.

* pm2, forever 같은 프로세스 관리 툴을 사용해 프로세스를 띄운다
* watchdog같은 것을 활용하여 파일 작업의 동기화를 안정화시킨다
* pdf변환은 adobe api를 사용하지 않고 libreoffice의 unoserver를 활용한다.



### 전제사항

언어는 계속 nodejs + javascript를 사용한다. excelJS의 기능을 그대로 쓰기 위해서이다. 



### 관문 (`index.js`)

```javascript
const express = require('express');
const multer = require('multer');
const ExcelJS = require('exceljs');
const fs = require('fs');
const path = require('path');

const app = express();
const port = 3000;

// 파일 저장을 위한 multer 설정
const storage = multer.diskStorage({
    destination: (req, file, cb) => {
        cb(null, 'uploads/');
    },
    filename: (req, file, cb) => {
        cb(null, req.body.email + '_' + Date.now() + '_' + file.originalname);
    }
});

const upload = multer({ storage: storage });

// 정적 파일 제공 설정 (HTML 포함)
app.use(express.static('public'));

app.post('/upload', upload.fields([
    { name: 'excelFile', maxCount: 1 },
    { name: 'signFile', maxCount: 1 }
]), async(req, res) => {

    if (!req.files) {
        return res.status(400).send('No file uploaded.');
    }

    try {
        const filePath = path.join(__dirname, 'uploads', req.files['excelFile'][0].filename);
        const signPath = path.join(__dirname, 'uploads', req.files['signFile'][0].filename);
        const modifiedFilePath = path.join(__dirname, 'readyfiles', 'modified_' + req.files['excelFile'][0].filename);
        const workbook = new ExcelJS.Workbook();
        await workbook.xlsx.readFile(filePath);

        const sizeOf =require('image-size');
        var dimension = sizeOf(signPath);

        //const worksheet = workbook.getWorksheet('Sheet1'); // 첫 번째 워크시트 선택
        const worksheet = workbook.getWorksheet(1); // 첫 번째 워크시트 선택


        // 이미지 추가
        const imageId = workbook.addImage({
            buffer: fs.readFileSync(signPath), // 이미지 경로 변경 필요
            extension: 'png',
        });

        //J4 Cell
        const colIndex = 9;
        const rowIndex = 3;


        worksheet.addImage(imageId,{
            tl: {col: colIndex+0.999, row:rowIndex},
            ext: {width:dimension.width, height:dimension.height},
            editAs: 'absolute'
        } );


        // 수정된 파일 저장
        //await workbook.xlsx.writeFile(modifiedFilePath);
        const buffer = await workbook.xlsx.writeBuffer();
        fs.writeFileSync(modifiedFilePath, buffer);
	
        res.sendFile(path.join(__dirname, 'public', 'end.html'));
    } catch (error) {
        console.error(error);
        //res.status(500).send('An error occurred.');
        res.status(500).sendFile(path.join(__dirname, 'public', 'xlsxerror.html'))
    }

});

// 정적 파일 제공 (업로드 폴더)
app.use('/uploads', express.static('uploads'));

app.listen(port, () => {
    console.log(`Server running at http://localhost:${port}/`);
});
```



public폴더의 `index.html`파일로부터 이미지와 엑셀 파일을 업로드 하면 express 서버에서 upload파일로 떨군 뒤 이미지를 엑셀 파일에 삽입하고 readyfiles 폴더에 저장하는 로직이다. 

힘들었던 부분은 이미지 처리 파일의 저장 부분이었다. 처음에는 그냥 `workbook.xlsx.writeFile`을 사용했는데, 뒤에 나오겠지만 파일이 올라오면 바로 작업을 하고 readyfiles로 저장한다. 그럼 다른 와치독프로그램이 이 폴더만 보고 있다가 파일이 생기면 바로 unoserver로 보내 pdf 변환작업에 들어간다. 

그런데 문제가 있었다. 와치독 프로그램이 파일을 감지하여 해당 파일을 pdf 변환을 위한 unoserver에 보내는데 결과는 항상 빈 백지만 나오는 것이었다. readyfiles에 저장된 파일에 문제가 있다는 증거였다. 신기한것이 나중에 readyfiles에 생성된 파일들을 보면 문제는 없었다.

원인은 와치독프로그램이 캐치한 파일의 당시 상태가 완료된 파일이 아니라는 것이었다(몇 ms차이이지만). 찾아보니 `xlsx.writeFile`은 파일을 비동기로 작성하기 때문에 파일을 일단 readyfiles 폴더에 빈 파일로 생성후, 작성을 이어가는 방식이었다. 와치독은 이 빈 파일을 가져다가 작업을 하였기에 계속 백지 pdf파일이 생성되는 것이었다.

이에 작업파일을 일단 버퍼에 저장을 하고 작업이 완료되면 버퍼를 한번에 `writeFileSync`를 하는 방식으로 바꾸었다.

```javascript
        // 수정된 파일 저장
        //await workbook.xlsx.writeFile(modifiedFilePath);
        const buffer = await workbook.xlsx.writeBuffer();
        fs.writeFileSync(modifiedFilePath, buffer);
```



xlsx파일의 pdf 변환작업을 위한 unoserver는 도커 컨테이너로 띄웠다. 컨테이너 작업은 https://hub.docker.com/r/libreofficedocker/libreoffice-unoserver 를 참조하기 바란다.

이제 readyfiles폴더를 감시하면서 파일이 생성되면 이를 가지고 pdf변환을 시키는 와치독 프로그램을 만들어야 한다. 프로그램은 다음과 같다. (transxlsx2pdf.js)



### pdf변환 : `transxlsx2pdf.js`



```javascript
//Watch working directory and make change xlsx files to pdf on the fly.
//Use Unoserver for transforming xlsx to pdf 
const path = require('path');
const axios = require('axios');
const FormData = require('form-data');
const fs = require('fs');
const chokidar = require('chokidar');

// 감시할 디렉토리 지정
const directoryToWatch = path.join(__dirname, 'readyfiles');
//pdf작업 디렉토리 지정
const pdfDirectory = path.join(__dirname,'pdffiles');

async function sendPostRequest(xlsxfilename){
    const url = 'http://127.0.0.1:2004/request';
    const formData = new FormData();

    // 파일 추가
    formData.append('file', fs.createReadStream(xlsxfilename));
    // 추가 폼 필드
    formData.append('convert-to', 'pdf');

    try {
      const response = await axios.post(url, formData, {
          headers: {
              ...formData.getHeaders(),
          },
          responseType: 'arraybuffer' // 파일로 다운로드 받기 위해 필요
      });

      // 파일로 저장
      fs.writeFileSync(pdfDirectory + '/' + path.basename(xlsxfilename) + '.pdf', response.data);

      console.log('File downloaded and saved');
    }catch (error) {
          console.error('Error during the request:', error.message);
    }
}

// chokidar 감시자 설정
const watcher = chokidar.watch(directoryToWatch, {
  ignored: /(^|[\/\\])\../, // 숨김 파일 무시
  persistent: true
});
  

// 파일이 추가되었을 때의 이벤트 핸들러
watcher.on('add', path => {sendPostRequest(path);console.log(`File ${path} has been converted.`)});


// 에러 핸들링
watcher.on('error', error => console.log(`Watcher error: ${error}`));

console.log(`Now watching for file changes in ${directoryToWatch}`);


```



* `const chokidar = require('chokidar');` : nodejs에서 와치독 기능을 하는 라이브러리(`chokidar`)이다. 
* `    const url = 'http://127.0.0.1:2004/request';`: 이 url은 도커 컨테이너로 띄운 unoserver의 url이다. 내부에서만 사용할 것이며 이 서버에 xlsx파일을 `axios`를 사용해 전달하고 pdf결과를 `response.data`로 받을 것이다.
* `const watcher = chokidar.watch()`: 감시자 설정 
* `watcher.on()` : 파일이 추가 되었을때 이벤트 핸들러. 여기서는 `add` 및 `error`에 대해 사용했다. 



pdf 파일은 pdffiles라는 폴더에 저장이 되게 된다. 이 pdffiles 폴더에 파일이 생기면 이 파일을 입력된 이메일 주소로 보내주는 와치독 프로그램을 하나 더 만들었다. (mailing.js)



### 메일발송(`mailing.js`)

```javascript
//Watch working directory and make change xlsx files to pdf on the fly.
//Use Unoserver for transforming xlsx to pdf 
const path = require('path');
const fs = require('fs');
const chokidar = require('');
const nodemailer = require('nodemailer');


// 감시할 디렉토리 지정
const directoryToWatch = path.join(__dirname, 'pdffiles');
// 메일 본문


const htmlContent = fs.readFileSync(path.join(__dirname, 'public', 'emailbody.html'), 'utf8');

function extractEmailFromFilename(filename) {
  // '_'를 구분자로 사용하여 파일 이름을 분리합니다.
  const parts = filename.split('_');
  
  // 두 번째 이메일 정보(두 번째 요소)를 추출합니다.
  if (parts.length > 1) {
    return parts[1];
  } else {
    throw new Error('파일 이름에서 두 번째 이메일 정보를 찾을 수 없습니다.');
  }
}

async function sendmail(pdffilename){
    const targetEmailAddr = extractEmailFromFilename(path.basename(pdffilename));

    let transporter = nodemailer.createTransport(
      {
        service: 'gmail',
        auth: {
          user: "logicalxxx@gmail.com",
          pass: "yourpassword",
        }
      }
    );
    console.log(htmlContent);
    let mailOption = {
      from: "logicalandemotional@gmail.com",
      to : targetEmailAddr,
      subject: "정산 pdf 메일",
      html: htmlContent,
      attachments: [
        {
          filename: path.basename(pdffilename),
          path: pdffilename
        }
      ]
    }

    transporter.sendMail(mailOption, function(error, info){
      if(error) {
        console.log(error);
      }
      else{
        console.log('Email sent:' + info.response);
      }
    }); 
    
}

// chokidar 감시자 설정
const watcher = chokidar.watch(directoryToWatch, {
  ignored: /(^|[\/\\])\../, // 숨김 파일 무시
  persistent: true
});
  

// 파일이 추가되었을 때의 이벤트 핸들러
watcher.on('add', path => {sendmail(path);console.log(`File ${path} has been sent`)});


// 에러 핸들링
watcher.on('error', error => console.log(`Watcher error: ${error}`));

console.log(`Now watching for file changes in ${directoryToWatch}`);
```



여기에서는 크게 어려운 부분은 없었다. 다만 gmail을 smpt서버로 활용하기 위해서는 예전에는 `smpt.gmail.com`을 쓴 기억이 있는데 `nodemailer`에서는 그냥 다음과 같이 작성하는 것으로 바뀌었다.

```javascript
        service: 'gmail',
        auth: {
          user: "logicalxxx@gmail.com",
          pass: "yourpassword",
        }
```

이때 `pass` 부분은 gmail패스워드가 아닌 gmail의 '앱 비밀번호'를 넣어야 한다. 자세한 것은 gmail의 앱 비밀번호 설정을 참조하기 바란다. 



프로세스는 이번에는 nohub으로 띄우지 않고 다음을 사용해 띄운다. 

* index.js : nginx unit에 설정하여 띄움
* transxlsx2pdf.js, mailing.js : forever를 사용하여 띄움



<h3 style='text-align:center'>끝</h3>



