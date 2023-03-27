---
title: (Jekyll) [Github 블로그, chirpy 테마] 지킬에 글쓰기
author: jun
date: 2019-08-08 14:10:00 +0800
categories: [Jekyll, Tutorial]
tags: [jekyll, github]
render_with_liquid: false
comments: true
---

<span style="color:grey">[Jekyll(chirpy theme) write-a-new-post.md](https://github.com/cotes2020/jekyll-theme-chirpy/blob/master/_posts/2019-08-08-write-a-new-post.md) 해당 글을 번역 및 요약하여 작성한 포스트 입니다.</span>

## 포스트 네이밍과 경로 (Naming and Path)

`_posts`{: .filepath} 디렉토리 아래에 `YYYY-MM-DD-TITLE.EXTENSION`{: .filepath} 형식의 파일을 생성합니다.
`EXTENSION`{: .filepath}은 반드시 `md`{: .filepath}나 `markdown`{: .filepath}중 하나여야 합니다.

## Front Matter

글의 최상단에서는 다음과 같이 시작합니다. ([Front Matter](https://jekyllrb.com/docs/front-matter/))

```yaml
---
title: TITLE
date: YYYY-MM-DD HH:MM:SS +/-TTTT
categories: [TOP_CATEGORIE, SUB_CATEGORIE]
tags: [TAG]     # TAG names should always be lowercase
---
```

> The posts' _layout_ has been set to `post` by default, so there is no need to add the variable _layout_ in the Front Matter block.
> {: .prompt-tip }

### 타임존 (Timezone of Date)

포스트 릴리즈 시간을 정확하게 기록하기 위해서, `_config.yml`{: .filepath} 파일에서 타임존(timezone: Asia/Seoul)을 세팅하고 Front Matter의 `date` Format(`+/-TTTT`, e.g. `+0800`.)을 설정하여 포스팅합니다.

### 카테고리와 태그 (Categories and Tags)

 `categories` 는 최대 2개까지 설정할 수 있으며, `tags` 는 0~제한없음 까지 설정할 수 있습니다. (tag는 항상 소문자로 작성되야 합니다)

```yaml
---
categories: [Animal, Insect]
tags: [bee]
---
```

### 작성자 정보 (Author Information)

게시물의 작성자 정보는 일반적으로 Front Matter 에 채울 필요가 없으며 기본적으로 `_config.yml`의 `social.name`와 `social.links`에서 가져옵니다. 그러나 다음과 같이 재정의할 수도 있습니다.

`_data/authors.yml`에 작성자 정보 추가(해당 파일이 없다면 생성해줍니다)

```yaml
<author_id>:
  name: <full name>
  twitter: <twitter_of_author>
  url: <homepage_of_author>
```

{: file="_data/authors.yml" }

그리고나서, [Front Matter](https://jekyllrb.com/docs/front-matter/)에 다음과 같이 작성할 수 있습니다.

```yaml
---
author: <author_id>                     # for single entry
# or
authors: [<author1_id>, <author2_id>]   # for multiple entries
---
```

## Table of Contents

**T**able **o**f **C**ontents (TOC)를 비활성화하기 위해서는 `_config.yml`{: .filepath} 에서 toc 속성을 false로 설정합니다. (기본은 true)

특정 포스트에서만 비활성화 하려면 [Front Matter](https://jekyllrb.com/docs/front-matter/) 에서 다음과 같이 설정합니다.

```yaml
---
toc: false
---
```

## 이미지 (Images)

### 이미지 캡션 (Caption)

이미지 캡션을 설정할 수 있습니다.

```markdown
![img-description](/path/to/image)
_Image Caption_
```

{: .nolineno}

### 이미지 크기 (Size)

이미지의 높이와 너비를 설정할 수 있습니다.

```markdown
![Desktop View](/assets/img/sample/mockup.png){: width="700" height="400" }
```

{: .nolineno}

Chirpy v5.0.0 부터 높이 및 너비에 대해 약어를 지원합니다(높이 → h, 너비 → w)

```markdown
![Desktop View](/assets/img/sample/mockup.png){: w="700" h="400" }
```

{: .nolineno}

### 이미지 위치 (Position)

기본적으로 이미지는 중앙에 위치하지만 `normal`, `left` 및 `right` 클래스 중 하나를 사용하여 위치를 지정할 수 있습니다.

> Once the position is specified, the image caption should not be added.
> {: .prompt-warning }

- **Normal position**

  Image will be left aligned in below sample:

  ```markdown
  ![Desktop View](/assets/img/sample/mockup.png){: .normal }
  ```

  {: .nolineno}

- **Float to the left**

  ```markdown
  ![Desktop View](/assets/img/sample/mockup.png){: .left }
  ```

  {: .nolineno}

- **Float to the right**

  ```markdown
  ![Desktop View](/assets/img/sample/mockup.png){: .right }
  ```

  {: .nolineno}

### CDN URL

CDN에서 이미지를 호스팅하는 경우 `_config.yml`{: .filepath} 파일의 `img_cdn` 변수를 할당하여 CDN URL 중복작성을 피할 수 있습니다.

```yaml
img_cdn: https://cdn.com
```

{: file='_config.yml' .nolineno}



`img_cdn`이 할당되면 `/`로 시작하는 모든 이미지(사이트 아바타 및 게시물 이미지)의 경로에 CDN URL이 추가됩니다.

예를 들어, 이미지를 다음과 같이 설정했을 때,

```markdown
![The flower](/path/to/flower.png)
```

{: .nolineno}



파싱된 결과는 다음과 같이 CDN prefix인 `https://cdn.com` 이 자동으로 이미지경로에 추가됩니다.

```html
<img src="https://cdn.com/path/to/flower.png" alt="The flower">
```

{: .nolineno }

### 이미지 경로 (Image Path)

게시물에 많은 이미지가 포함되어 있는 경우 이미지의 경로를 반복적으로 정의하는 것은 시간이 많이 걸리는 작업입니다. 이를 해결하기 위해 게시물의 YAML 블록에서 이 경로를 정의할 수 있습니다.

```yml
---
img_path: /img/path/
---
```

다음과 같이 파일이름만 작성하면 됩니다.

```md
![The flower](flower.png)
```

{: .nolineno }

결과는 다음과 같습니다.

```html
<img src="/img/path/flower.png" alt="The flower">
```

{: .nolineno }

### 이미지 미리보기 (Preview Image)

게시물 상단에 이미지를 추가하고 싶다면 `1200 x 630` 해상도의 이미지를 제공해주세요. 이미지 화면 비율이 `1.91 : 1`을 충족하지 않으면 이미지 크기가 조정되어 잘립니다.

이러한 전제 조건을 알고 있으면 이미지 속성 설정을 시작할 수 있습니다.

```yaml
---
image:
  path: /path/to/image
  alt: image alternative text
---
```

[`img_path`](#image-path)도 미리보기 이미지로 전달할 수 있습니다. 즉, 속성 `path`에는 이미지 파일 이름만 있으면 됩니다.간단한 사용을 위해 `이미지`를 사용하여 경로를 정의할 수도 있습니다.

```yml
---
image: /path/to/image
---
```

## 포스트 고정 (Pinned Posts)

하나 이상의 게시물을 홈페이지 상단에 고정할 수 있으며 고정된 게시물은 게시 날짜에 따라 역순으로 정렬됩니다.

```yaml
---
pin: true
---
```

## 프롬프트 (Prompts)

프롬프트에는 `tip`, `info`, `warning` 및 `danger`와 같은 여러 유형이 있습니다. blockquote에 `prompt-{type}` 클래스를 추가하여 생성할 수 있습니다. <br/>예를 들어 다음과 같이 '정보' 유형의 프롬프트를 정의합니다.

```md
> Example line for prompt.
{: .prompt-info }
```

{: .nolineno }

## 문법 (Syntax)

### 인라인 코드 (Inline Code)

```md
`inline code part`
```

{: .nolineno }

### 파일경로 강조 (Filepath Hightlight)

```md
`/path/to/a/file.extend`{: .filepath}
```

{: .nolineno }

### 코드블록 (Code Block)

마크다운 기호 ```` ``` ````는 다음과 같이 코드 블록을 생성할 수 있습니다.

```md
​```
This is a plaintext code snippet.
​```
```

#### 언어 설정 (Specifying Language)

```` ```{language} ````를 사용하면 구문 강조 표시가 있는 코드 블록을 생성할 수 있습니다.

```markdown
​```yaml
key: value
​```
```

> The Jekyll tag `{% highlight %}` is not compatible with this theme.
> {: .prompt-danger }

#### 줄 번호 (Line Number)

기본적으로 `일반 텍스트`, `콘솔` 및 `터미널`을 제외한 모든 언어는 줄 번호를 표시합니다. 코드 블록의 줄 번호를 숨기려면 `nolineno` 클래스를 추가합니다.

```markdown
​```shell
echo 'No more line numbers!'
​```
{: .nolineno }
```

## 동영상 (Videos)

다음과 같이 동영상을 삽입할 수 있습니다.

```liquid
{% include embed/{Platform}.html id='{ID}' %}
```

여기서 `Platform`은 플랫폼 이름의 소문자이고 `ID`는 비디오 ID입니다. 다음 표는 주어진 동영상 URL에서 필요한 두 개의 매개변수를 얻는 방법을 보여주며 현재 지원되는 동영상 플랫폼도 알 수 있습니다.

| Video URL                                | Platform  | ID            |
| ---------------------------------------- | --------- | :------------ |
| [https://www.**youtube**.com/watch?v=**H-B46URT4mg**](https://www.youtube.com/watch?v=H-B46URT4mg) | `youtube` | `H-B46URT4mg` |
| [https://www.**twitch**.tv/videos/**1634779211**](https://www.twitch.tv/videos/1634779211) | `twitch`  | `1634779211`  |



## Learn More

[Jekyll Docs: Posts](https://jekyllrb.com/docs/posts/).