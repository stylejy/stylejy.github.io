# Like Tech 블로그 Posting 방법

## 구조

`블로그 포스팅을 위한 구조만 나타내겠습니다. 그외의 상세 구조는 추후 기타 란에 업데이트 하겠습니다.`

알아둬야 할 폴더는 아래 두개를 참고해야 합니다.

- \_posts
- assets/img

위의 두 폴더에 파일을 업로드 하고 push를 하면 웹페이지에서 포스트가 자동으로 출력 됩니다.

## Post 파일 생성

Post 파일은 \_posts 폴더에 새파일을 생성한후 아래와 같은 양식으로 파일을 생성합니다.

`YYYY-MM-DD-{keyword1-keyworkd2...}.markdown`
예: 2020-10-25-named-parameter-failiure.markdown

## Post 파일 헤더 작성 방법

아래의 정보를 '---' 로 감싸진 곳에 post에 대한 정보를 작성해야 합니다.

- layout: post 로 고정 되고 이것은 바꾸지 않아야 합니다.
- title: 웹페이지에 표현될 용도로 간결하게 작성합니다.
- date: 현재 날짜를 기록합니다.
- img: 포스트롤 돋보이게 하고 싶은 경우 imgage 파일을 img폴더에 넣고 해당 파일을 기록합니다. 파일 크기에 대한 정보는 spyon-jest.jpg 를 참고 하시면 됩니다.
- tags: Array의 형태로 키워드를 자유롭게 입력 합니다.
- author: 처음에 한번만 본인의 이름으로 맞춰놓고 다음부터 포스트는 항상 갖게 사용하여야 합니다. 그래야 본인의 포스트 밑에 본인의 이름이 출력 됩니다.
- author-pic: 본인의 프로필 이미지 jy.jpg, jh.jpg or sk.jpg 셋중에 하나를 입력 하고 이후 항상 갖게 사용하여야 본인의 포스트 밑에 본인의 프로필 이미지가 출력 됩니다.
- about-author: 본인의 포스트 밑에 본인 소개 문구를 넣고 싶으면 사용합니다.(optional)
- email: 본인의 이메일 주소를 포스트 밑에 넣고 싶은 경우 입력 합니다.(optional)

기존 포스트의 헤더를 참고 하시면 됩니다.

## Post 본문 작성

본문은 markdown 형식으로 작성합니다. 이미 올라와져 있는 post 를 참고 해서 작성하셔도 되고
옆의 참고 사이트를 통해서 사용방법 습득후 사용하실 수도 있습니다.

https://www.markdownguide.org/basic-syntax/
