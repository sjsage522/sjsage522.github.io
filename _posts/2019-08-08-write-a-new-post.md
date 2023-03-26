---
title: (Jekyll) [Github 블로그, chirpy 테마] 지킬에 글쓰기
author: jun
date: 2019-08-08 14:10:00 +0800
categories: [Jekyll, Tutorial]
tags: [jekyll, github]
render_with_liquid: false
---

<span style="color:grey">[Jekyll(chirpy theme) write-a-new-post.md](https://github.com/cotes2020/jekyll-theme-chirpy/blob/master/_posts/2019-08-08-write-a-new-post.md) 해당 글을 번역 및 요약하여 작성한 포스트 입니다.</span>

## Naming and Path

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

### Timezone of Date

포스트 릴리즈 시간을 정확하게 기록하기 위해서, `_config.yml`{: .filepath} 파일에서 타임존(timezone: Asia/Seoul)을 세팅하고 Front Matter의 `date` Format(`+/-TTTT`, e.g. `+0800`.)을 설정하여 포스팅합니다.

### Categories and Tags

 `categories` 는 최대 2개까지 설정할 수 있으며, `tags` 는 0~제한없음 까지 설정할 수 있습니다. (tag는 항상 소문자로 작성되야 합니다)

```yaml
---
categories: [Animal, Insect]
tags: [bee]
---
```

### Author Information

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

By default, the **T**able **o**f **C**ontents (TOC) is displayed on the right panel of the post. If you want to turn it off globally, go to `_config.yml`{: .filepath} and set the value of variable `toc` to `false`. If you want to turn off TOC for a specific post, add the following to the post's [Front Matter](https://jekyllrb.com/docs/front-matter/):

```yaml
---
toc: false
---
```

## Comments

The global switch of comments is defined by variable `comments.active` in the file `_config.yml`{: .filepath}. After selecting a comment system for this variable, comments will be turned on for all posts.

If you want to close the comment for a specific post, add the following to the **Front Matter** of the post:

```yaml
---
comments: false
---
```

## Mathematics

For website performance reasons, the mathematical feature won't be loaded by default. But it can be enabled by:

```yaml
---
math: true
---
```

## Mermaid

[**Mermaid**](https://github.com/mermaid-js/mermaid) is a great diagrams generation tool. To enable it on your post, add the following to the YAML block:

```yaml
---
mermaid: true
---
```

Then you can use it like other markdown languages: surround the graph code with ```` ```mermaid ```` and ```` ``` ````.

## Images

### Caption

Add italics to the next line of an image，then it will become the caption and appear at the bottom of the image:

```markdown
![img-description](/path/to/image)
_Image Caption_
```

{: .nolineno}

### Size

In order to prevent the page content layout from shifting when the image is loaded, we should set the width and height for each image.

```markdown
![Desktop View](/assets/img/sample/mockup.png){: width="700" height="400" }
```

{: .nolineno}

> For an SVG, you have to at least specify its _width_, otherwise it won't be rendered.
> {: .prompt-info }

Starting from _Chirpy v5.0.0_, `height` and `width` support abbreviations (`height` → `h`, `width` → `w`). The following example has the same effect as the above:

```markdown
![Desktop View](/assets/img/sample/mockup.png){: w="700" h="400" }
```

{: .nolineno}

### Position

By default, the image is centered, but you can specify the position by using one of the classes `normal`, `left`, and `right`.

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

### Dark/Light mode

You can make images follow theme preferences in dark/light mode. This requires you to prepare two images, one for dark mode and one for light mode, and then assign them a specific class (`dark` or `light`):

```markdown
![Light mode only](/path/to/light-mode.png){: .light }
![Dark mode only](/path/to/dark-mode.png){: .dark }
```

### Shadow

The screenshots of the program window can be considered to show the shadow effect:

```markdown
![Desktop View](/assets/img/sample/mockup.png){: .shadow }
```

{: .nolineno}

### CDN URL

If you host the images on the CDN, you can save the time of repeatedly writing the CDN URL by assigning the variable `img_cdn` of `_config.yml`{: .filepath} file:

```yaml
img_cdn: https://cdn.com
```

{: file='_config.yml' .nolineno}

Once `img_cdn` is assigned, the CDN URL will be added to the path of all images (images of site avatar and posts) starting with `/`.

For instance, when using images:

```markdown
![The flower](/path/to/flower.png)
```

{: .nolineno}

The parsing result will automatically add the CDN prefix `https://cdn.com` before the image path:

```html
<img src="https://cdn.com/path/to/flower.png" alt="The flower">
```

{: .nolineno }

### Image Path

When a post contains many images, it will be a time-consuming task to repeatedly define the path of the images. To solve this, we can define this path in the YAML block of the post:

```yml
---
img_path: /img/path/
---
```

And then, the image source of Markdown can write the file name directly:

```md
![The flower](flower.png)
```

{: .nolineno }

The output will be:

```html
<img src="/img/path/flower.png" alt="The flower">
```

{: .nolineno }

### Preview Image

If you want to add an image at the top of the post, please provide an image with a resolution of `1200 x 630`. Please note that if the image aspect ratio does not meet `1.91 : 1`, the image will be scaled and cropped.

Knowing these prerequisites, you can start setting the image's attribute:

```yaml
---
image:
  path: /path/to/image
  alt: image alternative text
---
```

Note that the [`img_path`](#image-path) can also be passed to the preview image, that is, when it has been set, the  attribute `path` only needs the image file name.

For simple use, you can also just use `image` to define the path.

```yml
---
image: /path/to/image
---
```

### LQIP

For preview images:

```yaml
---
image:
  lqip: /path/to/lqip-file # or base64 URI
---
```

> You can observe LQIP in the preview image of post [_Text and Typography_](/posts/text-and-typography/).

For normal images:

```markdown
![Image description](/path/to/image){: lqip="/path/to/lqip-file" }
```

{: .nolineno }

## Pinned Posts

You can pin one or more posts to the top of the home page, and the fixed posts are sorted in reverse order according to their release date. Enable by:

```yaml
---
pin: true
---
```

## Prompts

There are several types of prompts: `tip`, `info`, `warning`, and `danger`. They can be generated by adding the class `prompt-{type}` to the blockquote. For example, define a prompt of type `info` as follows:

```md
> Example line for prompt.
{: .prompt-info }
```

{: .nolineno }

## Syntax

### Inline Code

```md
`inline code part`
```

{: .nolineno }

### Filepath Hightlight

```md
`/path/to/a/file.extend`{: .filepath}
```

{: .nolineno }

### Code Block

Markdown symbols ```` ``` ```` can easily create a code block as follows:

```md
​```
This is a plaintext code snippet.
​```
```

#### Specifying Language

Using ```` ```{language} ```` you will get a code block with syntax highlight:

```markdown
​```yaml
key: value
​```
```

> The Jekyll tag `{% highlight %}` is not compatible with this theme.
> {: .prompt-danger }

#### Line Number

By default, all languages except `plaintext`, `console`, and `terminal` will display line numbers. When you want to hide the line number of a code block, add the class `nolineno` to it:

```markdown
​```shell
echo 'No more line numbers!'
​```
{: .nolineno }
```

#### Specifying the Filename

You may have noticed that the code language will be displayed at the top of the code block. If you want to replace it with the file name, you can add the attribute `file` to achieve this:

```markdown
​```shell
# content
​```
{: file="path/to/file" }
```

#### Liquid Codes

If you want to display the **Liquid** snippet, surround the liquid code with `{% raw %}` and `{% endraw %}`:

```markdown
{% raw %}
​```liquid
{% if product.title contains 'Pack' %}
  This product's title contains the word Pack.
{% endif %}
​```
{% endraw %}
```

Or adding `render_with_liquid: false` (Requires Jekyll 4.0 or higher) to the post's YAML block.

## Videos

You can embed a video with the following syntax:

```liquid
{% include embed/{Platform}.html id='{ID}' %}
```

Where `Platform` is the lowercase of the platform name, and `ID` is the video ID.

The following table shows how to get the two parameters we need in a given video URL, and you can also know the currently supported video platforms.

| Video URL                                | Platform  | ID            |
| ---------------------------------------- | --------- | :------------ |
| [https://www.**youtube**.com/watch?v=**H-B46URT4mg**](https://www.youtube.com/watch?v=H-B46URT4mg) | `youtube` | `H-B46URT4mg` |
| [https://www.**twitch**.tv/videos/**1634779211**](https://www.twitch.tv/videos/1634779211) | `twitch`  | `1634779211`  |



## Learn More

For more knowledge about Jekyll posts, visit the [Jekyll Docs: Posts](https://jekyllrb.com/docs/posts/).