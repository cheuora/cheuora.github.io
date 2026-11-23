# Thinking Storage of CHEUORA

소프트웨어 개발과 품질, AI, 교육, 독서 등 다양한 주제의 생각과 기록을 모으는 CHEUORA의 개인 블로그입니다. Jekyll과 Minima 테마를 기반으로 만들었으며, GitHub Actions를 통해 GitHub Pages에 배포합니다.

- 블로그: <https://cheuora.github.io/>
- 소개: [about.markdown](about.markdown)

## 기술 구성

- Jekyll `~> 4.3.2`, Minima `~> 2.5`
- Markdown, Liquid 템플릿, Sass
- `jekyll-feed`: RSS 피드
- `jekyll-paginate`: 목록 페이지당 글 5개 표시
- Utterances: GitHub Issues 기반 댓글
- MathJax: `use_math: true`로 지정한 페이지의 수식 표시

## 디렉토리 구조

```text
.
├── _posts/                  # 연도별 Markdown 게시글과 이미지
├── _layouts/                # 홈, 게시글, 일반 페이지, 태그 페이지 레이아웃
├── _includes/               # 헤더, 푸터, 댓글 등 공통 구성 요소
├── _sass/                   # 스타일 정의
├── assets/                  # 스타일 진입점과 공용 이미지
├── tag/                     # 스크립트로 생성하는 태그별 페이지
├── .github/workflows/       # GitHub Pages 빌드·배포 워크플로
├── _config.yml              # 사이트 설정
├── Gemfile                  # Ruby 의존성
├── Gemfile.lock             # 의존성 버전 잠금
├── index.html               # 블로그 홈
├── about.markdown           # 소개 페이지
├── tagarchive.markdown      # 태그 목록 페이지
├── tag_collect.py           # 태그 페이지 생성 스크립트
└── localrun.sh              # 로컬 서버 실행 스크립트
```

## 로컬 실행

Ruby와 Bundler가 필요합니다. 저장소의 배포 워크플로는 Ruby 3.1을 사용하며, `Gemfile.lock`의 Bundler 버전은 2.4.22입니다. 태그 페이지를 생성하려면 Python 3도 필요합니다.

저장소 루트에서 다음 명령을 실행합니다.

```sh
bundle install
bundle exec jekyll serve
```

브라우저에서 <http://localhost:4000>에 접속합니다. Bash 환경에서는 `bash localrun.sh`로도 서버를 실행할 수 있습니다. 종료하려면 `Ctrl+C`를 누릅니다.

`_config.yml`을 수정한 경우 서버를 다시 시작해야 반영됩니다.

정적 파일만 빌드하려면 다음 명령을 사용합니다. 결과는 `_site/`에 생성되며 Git 추적에서 제외됩니다.

```sh
bundle exec jekyll build
```

## 글 작성

1. `_posts/연도/` 아래에 `YYYY-MM-DD-제목.md` 형식의 파일을 만듭니다.
2. 파일 맨 위에 YAML front matter를 작성하고, 그 아래에 Markdown 본문을 추가합니다.

예: `_posts/2026/2026-11-04-new-post.md`

```markdown
---
layout: post
title: "새 글 제목"
tags: [AI, QA]
---

여기에 본문을 작성합니다.
```

수식이 필요한 글은 front matter에 `use_math: true`를 추가합니다. 공용 이미지는 `assets/img/`에 두고 다음처럼 참조할 수 있습니다.

```markdown
![이미지 설명](/assets/img/example.png)
```

## 태그 페이지 갱신

게시글에 태그를 추가하거나 변경한 뒤 저장소 루트에서 실행합니다.

```sh
python tag_collect.py
```

스크립트는 `_posts/` 하위의 `.md` 게시글에서 태그를 수집하고, **기존 `tag/*.md` 파일을 삭제한 뒤 다시 생성**합니다. 태그 페이지는 직접 편집하지 않고 게시글의 `tags`를 수정한 뒤 재생성합니다. 태그는 위 예시처럼 한 줄의 배열 형식으로 작성하는 것이 좋습니다.

생성된 `tag/` 변경 사항도 게시글과 함께 커밋합니다. 현재 배포 워크플로는 태그 생성 스크립트를 자동 실행하지 않습니다.

## 배포

[`.github/workflows/jekyll.yml`](.github/workflows/jekyll.yml)에 배포 과정이 정의되어 있습니다.

1. `main` 브랜치에 push하면 워크플로가 실행됩니다. GitHub Actions 화면에서 수동 실행도 가능합니다.
2. Ruby 의존성을 설치하고 `JEKYLL_ENV=production`으로 Jekyll을 빌드합니다.
3. `_site/` 결과물을 GitHub Pages에 배포합니다.

배포 상태와 오류는 저장소의 **Actions → Deploy Jekyll site to Pages**에서 확인합니다.

## 주요 수정 위치

| 수정할 내용 | 파일 또는 디렉토리 |
| --- | --- |
| 사이트 제목, 작성자, 시간대, 페이지당 글 수 | `_config.yml` |
| 소개 페이지 | `about.markdown` |
| 홈의 글 목록 | `_layouts/home.html` |
| 게시글 화면과 Utterances 댓글 설정 | `_layouts/post.html` |
| 헤더와 푸터 | `_includes/header.html`, `_includes/footer.html` |
| 스타일 | `assets/main.scss`, `_sass/` |
| 태그 목록과 태그별 화면 | `tagarchive.markdown`, `_layouts/tagpage.html` |
