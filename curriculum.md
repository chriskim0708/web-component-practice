# Web Components 학습 커리큘럼

## 목차
1. [Web Components 소개](#1-web-components-소개)
2. [Custom Elements 기초](#2-custom-elements-기초)
3. [Shadow DOM 이해하기](#3-shadow-dom-이해하기)
4. [HTML Templates & Slots](#4-html-templates--slots)
5. [실전 프로젝트](#5-실전-프로젝트)

---

## 1. Web Components 소개

### 개념
**Web Components**는 재사용 가능한 커스텀 HTML 요소를 만들기 위한 웹 표준 기술의 집합입니다. 프레임워크 없이 순수 JavaScript로 캡슐화된 컴포넌트를 만들 수 있습니다.

### 핵심 기술 3가지
1. **Custom Elements** - 사용자 정의 HTML 태그
2. **Shadow DOM** - 스타일과 마크업 캡슐화
3. **HTML Templates** - 재사용 가능한 마크업 템플릿

### 예제 1-1: 간단한 Hello World 컴포넌트

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>Web Component 첫 예제</title>
</head>
<body>
    <!-- 사용자 정의 요소 사용 -->
    <hello-world></hello-world>

    <script>
        // Custom Element 정의
        class HelloWorld extends HTMLElement {
            constructor() {
                super();
                this.innerHTML = '<h1>안녕하세요, Web Components!</h1>';
            }
        }

        // 커스텀 요소 등록
        customElements.define('hello-world', HelloWorld);
    </script>
</body>
</html>
```

**실습 목표:**
- Custom Element의 기본 구조 이해
- `customElements.define()` 사용법 익히기

---

## 2. Custom Elements 기초

### 용어 설명

#### **Custom Elements (커스텀 엘리먼트)**
표준 HTML 요소를 확장하거나 완전히 새로운 HTML 요소를 정의할 수 있는 JavaScript API입니다.

**규칙:**
- 이름에 하이픈(`-`)이 반드시 포함되어야 함 (예: `my-button`, `user-card`)
- 소문자로 작성
- HTML 표준 요소와 이름 충돌 방지

#### **Lifecycle Callbacks (생명주기 콜백)**
Custom Element가 DOM에 추가/제거되거나 변경될 때 자동으로 호출되는 메서드들:

- `connectedCallback()` - 요소가 DOM에 추가될 때
- `disconnectedCallback()` - 요소가 DOM에서 제거될 때
- `attributeChangedCallback()` - 속성이 변경될 때
- `adoptedCallback()` - 다른 문서로 이동될 때

### 예제 2-1: Lifecycle Callbacks 활용

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>Lifecycle 예제</title>
</head>
<body>
    <user-card name="홍길동" role="개발자"></user-card>

    <button id="toggle">카드 토글</button>

    <script>
        class UserCard extends HTMLElement {
            // 감시할 속성 정의
            static get observedAttributes() {
                return ['name', 'role'];
            }

            constructor() {
                super();
                console.log('생성자 호출');
            }

            // DOM에 추가될 때
            connectedCallback() {
                console.log('DOM에 추가됨');
                this.render();
            }

            // DOM에서 제거될 때
            disconnectedCallback() {
                console.log('DOM에서 제거됨');
            }

            // 속성이 변경될 때
            attributeChangedCallback(name, oldValue, newValue) {
                console.log(`속성 ${name}: ${oldValue} → ${newValue}`);
                if (oldValue !== newValue) {
                    this.render();
                }
            }

            render() {
                const name = this.getAttribute('name') || '이름 없음';
                const role = this.getAttribute('role') || '역할 없음';

                this.innerHTML = `
                    <div style="border: 2px solid #333; padding: 20px; margin: 10px;">
                        <h2>${name}</h2>
                        <p>역할: ${role}</p>
                    </div>
                `;
            }
        }

        customElements.define('user-card', UserCard);

        // 토글 버튼 기능
        const card = document.querySelector('user-card');
        const toggleBtn = document.getElementById('toggle');

        toggleBtn.addEventListener('click', () => {
            if (card.isConnected) {
                card.remove();
            } else {
                document.body.insertBefore(card, toggleBtn);
            }
        });

        // 3초 후 속성 변경
        setTimeout(() => {
            card.setAttribute('name', '김철수');
            card.setAttribute('role', '디자이너');
        }, 3000);
    </script>
</body>
</html>
```

**실습 목표:**
- Lifecycle callbacks 동작 확인
- 콘솔에서 로그 확인
- 속성 변경 반응성 이해

### 예제 2-2: 인터랙티브 카운터 컴포넌트

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>카운터 컴포넌트</title>
</head>
<body>
    <counter-button></counter-button>
    <counter-button initial="10"></counter-button>

    <script>
        class CounterButton extends HTMLElement {
            constructor() {
                super();
                this.count = parseInt(this.getAttribute('initial')) || 0;
            }

            connectedCallback() {
                this.render();
                this.querySelector('button').addEventListener('click', () => {
                    this.count++;
                    this.render();
                });
            }

            render() {
                this.innerHTML = `
                    <div style="margin: 10px;">
                        <button style="padding: 10px 20px; font-size: 16px;">
                            클릭 횟수: ${this.count}
                        </button>
                    </div>
                `;
            }
        }

        customElements.define('counter-button', CounterButton);
    </script>
</body>
</html>
```

**실습 목표:**
- 상태 관리 기초
- 이벤트 핸들링
- 속성을 통한 초기값 설정

---

## 3. Shadow DOM 이해하기

### 용어 설명

#### **Shadow DOM (섀도우 DOM)**
Web Component의 핵심 기술로, DOM과 스타일을 캡슐화하여 외부로부터 격리시키는 기술입니다.

**주요 개념:**
- **Shadow Host** - Shadow DOM이 연결되는 일반 DOM 노드
- **Shadow Root** - Shadow Tree의 최상위 노드
- **Shadow Tree** - Shadow DOM 내부의 DOM 트리
- **Light DOM** - 일반적인 DOM (Shadow DOM의 반대 개념)

**장점:**
- 스타일 격리 (외부 CSS 영향 차단)
- DOM 격리 (내부 구조 숨김)
- 이름 충돌 방지

#### **Shadow DOM Mode**
- `open` - JavaScript로 Shadow Root 접근 가능 (`element.shadowRoot`)
- `closed` - Shadow Root 접근 불가 (보안 강화)

### 예제 3-1: Shadow DOM 기본 사용

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>Shadow DOM 기초</title>
    <style>
        /* 이 스타일은 Shadow DOM 내부에 영향을 주지 않음 */
        p {
            color: red;
            font-size: 20px;
        }
    </style>
</head>
<body>
    <p>일반 DOM의 문단 (빨간색, 20px)</p>

    <shadow-card></shadow-card>

    <script>
        class ShadowCard extends HTMLElement {
            constructor() {
                super();

                // Shadow DOM 생성 (open mode)
                const shadow = this.attachShadow({ mode: 'open' });

                // Shadow DOM 내부 HTML
                shadow.innerHTML = `
                    <style>
                        /* 이 스타일은 Shadow DOM 내부에만 적용됨 */
                        p {
                            color: blue;
                            font-size: 14px;
                            padding: 20px;
                            border: 2px solid blue;
                        }
                    </style>
                    <p>Shadow DOM의 문단 (파란색, 14px, 테두리)</p>
                `;
            }
        }

        customElements.define('shadow-card', ShadowCard);

        // Shadow Root 접근 테스트
        const card = document.querySelector('shadow-card');
        console.log('Shadow Root:', card.shadowRoot);
    </script>
</body>
</html>
```

**실습 목표:**
- Shadow DOM 스타일 격리 확인
- `attachShadow()` 사용법
- open/closed mode 차이 이해

### 예제 3-2: 스타일이 적용된 프로필 카드

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>프로필 카드</title>
</head>
<body>
    <profile-card
        name="김개발"
        title="Frontend Developer"
        avatar="https://via.placeholder.com/100">
    </profile-card>

    <profile-card
        name="이디자인"
        title="UI/UX Designer"
        avatar="https://via.placeholder.com/100/ff6b6b">
    </profile-card>

    <script>
        class ProfileCard extends HTMLElement {
            constructor() {
                super();
                this.attachShadow({ mode: 'open' });
            }

            connectedCallback() {
                this.render();
            }

            render() {
                const name = this.getAttribute('name') || 'Unknown';
                const title = this.getAttribute('title') || 'No Title';
                const avatar = this.getAttribute('avatar') || '';

                this.shadowRoot.innerHTML = `
                    <style>
                        :host {
                            display: block;
                            margin: 20px;
                        }

                        .card {
                            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
                            border-radius: 15px;
                            padding: 30px;
                            color: white;
                            box-shadow: 0 10px 30px rgba(0,0,0,0.3);
                            max-width: 300px;
                            transition: transform 0.3s ease;
                        }

                        .card:hover {
                            transform: translateY(-5px);
                        }

                        .avatar {
                            width: 100px;
                            height: 100px;
                            border-radius: 50%;
                            border: 4px solid white;
                            margin-bottom: 15px;
                        }

                        h2 {
                            margin: 0 0 10px 0;
                            font-size: 24px;
                        }

                        p {
                            margin: 0;
                            opacity: 0.9;
                            font-size: 16px;
                        }
                    </style>

                    <div class="card">
                        <img class="avatar" src="${avatar}" alt="${name}">
                        <h2>${name}</h2>
                        <p>${title}</p>
                    </div>
                `;
            }
        }

        customElements.define('profile-card', ProfileCard);
    </script>
</body>
</html>
```

**실습 목표:**
- `:host` 선택자 사용법
- Shadow DOM 내부 스타일링
- CSS 변수와 gradient 활용

### 예제 3-3: CSS 변수를 통한 외부 스타일 제어

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>CSS 변수 활용</title>
    <style>
        /* 외부에서 CSS 변수로 Shadow DOM 스타일 제어 */
        custom-button {
            --button-bg: #3498db;
            --button-color: white;
            --button-hover: #2980b9;
        }

        custom-button.danger {
            --button-bg: #e74c3c;
            --button-hover: #c0392b;
        }

        custom-button.success {
            --button-bg: #2ecc71;
            --button-hover: #27ae60;
        }
    </style>
</head>
<body>
    <custom-button>기본 버튼</custom-button>
    <custom-button class="danger">위험 버튼</custom-button>
    <custom-button class="success">성공 버튼</custom-button>

    <script>
        class CustomButton extends HTMLElement {
            constructor() {
                super();
                this.attachShadow({ mode: 'open' });
            }

            connectedCallback() {
                this.shadowRoot.innerHTML = `
                    <style>
                        button {
                            background: var(--button-bg, #ccc);
                            color: var(--button-color, black);
                            border: none;
                            padding: 12px 24px;
                            font-size: 16px;
                            border-radius: 5px;
                            cursor: pointer;
                            transition: all 0.3s ease;
                        }

                        button:hover {
                            background: var(--button-hover, #999);
                            transform: scale(1.05);
                        }

                        button:active {
                            transform: scale(0.95);
                        }
                    </style>

                    <button><slot></slot></button>
                `;
            }
        }

        customElements.define('custom-button', CustomButton);
    </script>
</body>
</html>
```

**실습 목표:**
- CSS 변수를 통한 테마 제어
- Shadow DOM과 외부 스타일 연결
- `<slot>` 기본 사용법

---

## 4. HTML Templates & Slots

### 용어 설명

#### **`<template>` 요소**
렌더링되지 않는 재사용 가능한 HTML 마크업을 정의하는 요소입니다. JavaScript로 복제하여 사용합니다.

**특징:**
- 페이지 로드 시 렌더링되지 않음
- `content` 속성을 통해 접근
- `cloneNode(true)`로 복제하여 사용

#### **`<slot>` 요소**
외부에서 전달된 콘텐츠를 삽입할 수 있는 플레이스홀더입니다.

**종류:**
- **Default Slot** - 이름 없는 기본 슬롯
- **Named Slot** - `name` 속성으로 지정된 슬롯

**주요 개념:**
- **Light DOM** - 슬롯에 전달되는 외부 콘텐츠
- **Slot Fallback** - 콘텐츠가 없을 때 표시되는 기본값

### 예제 4-1: Template 활용

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>Template 예제</title>
</head>
<body>
    <!-- 재사용 가능한 템플릿 정의 -->
    <template id="card-template">
        <style>
            .card {
                border: 1px solid #ddd;
                border-radius: 8px;
                padding: 20px;
                margin: 10px;
                box-shadow: 0 2px 5px rgba(0,0,0,0.1);
            }
            .title {
                color: #333;
                font-size: 20px;
                margin: 0 0 10px 0;
            }
            .content {
                color: #666;
            }
        </style>
        <div class="card">
            <h3 class="title"></h3>
            <p class="content"></p>
        </div>
    </template>

    <div id="container"></div>

    <script>
        class TemplateCard extends HTMLElement {
            constructor() {
                super();
                this.attachShadow({ mode: 'open' });

                // 템플릿 복제
                const template = document.getElementById('card-template');
                const content = template.content.cloneNode(true);

                // 데이터 삽입
                content.querySelector('.title').textContent =
                    this.getAttribute('title') || '제목';
                content.querySelector('.content').textContent =
                    this.getAttribute('content') || '내용';

                this.shadowRoot.appendChild(content);
            }
        }

        customElements.define('template-card', TemplateCard);

        // 여러 카드 생성
        const container = document.getElementById('container');
        const cards = [
            { title: '첫 번째 카드', content: 'Template을 사용한 카드입니다.' },
            { title: '두 번째 카드', content: '코드 재사용이 쉽습니다.' },
            { title: '세 번째 카드', content: '유지보수가 편리합니다.' }
        ];

        cards.forEach(({ title, content }) => {
            const card = document.createElement('template-card');
            card.setAttribute('title', title);
            card.setAttribute('content', content);
            container.appendChild(card);
        });
    </script>
</body>
</html>
```

**실습 목표:**
- `<template>` 요소 사용법
- `content.cloneNode(true)` 이해
- 템플릿 기반 컴포넌트 생성

### 예제 4-2: Slot 기초 - Default Slot

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>Slot 기초</title>
</head>
<body>
    <fancy-box>
        <h2>슬롯에 전달된 제목</h2>
        <p>이 콘텐츠는 외부에서 전달되었습니다.</p>
    </fancy-box>

    <fancy-box>
        <!-- 콘텐츠 없음 - fallback 표시됨 -->
    </fancy-box>

    <script>
        class FancyBox extends HTMLElement {
            constructor() {
                super();
                this.attachShadow({ mode: 'open' });

                this.shadowRoot.innerHTML = `
                    <style>
                        .box {
                            border: 3px solid #9b59b6;
                            border-radius: 10px;
                            padding: 20px;
                            margin: 10px 0;
                            background: #f8f9fa;
                        }

                        ::slotted(h2) {
                            color: #9b59b6;
                            margin-top: 0;
                        }

                        ::slotted(p) {
                            color: #555;
                            line-height: 1.6;
                        }
                    </style>

                    <div class="box">
                        <slot>
                            <!-- Fallback 콘텐츠 -->
                            <p>여기에 콘텐츠를 넣어주세요!</p>
                        </slot>
                    </div>
                `;
            }
        }

        customElements.define('fancy-box', FancyBox);
    </script>
</body>
</html>
```

**실습 목표:**
- Default slot 사용법
- `::slotted()` 선택자 활용
- Fallback 콘텐츠 이해

### 예제 4-3: Named Slot - 복잡한 레이아웃

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>Named Slot</title>
</head>
<body>
    <article-card>
        <span slot="author">홍길동</span>
        <span slot="date">2024-01-15</span>
        <h2 slot="title">Web Components 완벽 가이드</h2>
        <div slot="content">
            <p>Web Components는 재사용 가능한 커스텀 요소를 만들 수 있는 강력한 기술입니다.</p>
            <p>Shadow DOM, Custom Elements, HTML Templates를 활용합니다.</p>
        </div>
        <div slot="footer">
            <button>좋아요</button>
            <button>공유하기</button>
        </div>
    </article-card>

    <script>
        class ArticleCard extends HTMLElement {
            constructor() {
                super();
                this.attachShadow({ mode: 'open' });

                this.shadowRoot.innerHTML = `
                    <style>
                        .card {
                            max-width: 600px;
                            margin: 20px auto;
                            border: 1px solid #e0e0e0;
                            border-radius: 12px;
                            overflow: hidden;
                            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
                        }

                        .header {
                            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
                            color: white;
                            padding: 30px;
                        }

                        .meta {
                            display: flex;
                            justify-content: space-between;
                            font-size: 14px;
                            margin-bottom: 15px;
                            opacity: 0.9;
                        }

                        ::slotted([slot="title"]) {
                            margin: 0;
                            font-size: 28px;
                        }

                        .content {
                            padding: 30px;
                            line-height: 1.8;
                            color: #333;
                        }

                        ::slotted([slot="content"]) {
                            margin: 0;
                        }

                        .footer {
                            padding: 20px 30px;
                            background: #f5f5f5;
                            border-top: 1px solid #e0e0e0;
                        }

                        ::slotted(button) {
                            padding: 10px 20px;
                            margin-right: 10px;
                            border: none;
                            border-radius: 5px;
                            background: #667eea;
                            color: white;
                            cursor: pointer;
                            transition: background 0.3s;
                        }

                        ::slotted(button:hover) {
                            background: #764ba2;
                        }
                    </style>

                    <div class="card">
                        <div class="header">
                            <div class="meta">
                                <span>작성자: <slot name="author">익명</slot></span>
                                <span>날짜: <slot name="date">날짜 없음</slot></span>
                            </div>
                            <slot name="title">
                                <h2>제목 없음</h2>
                            </slot>
                        </div>

                        <div class="content">
                            <slot name="content">
                                <p>내용이 없습니다.</p>
                            </slot>
                        </div>

                        <div class="footer">
                            <slot name="footer"></slot>
                        </div>
                    </div>
                `;
            }
        }

        customElements.define('article-card', ArticleCard);
    </script>
</body>
</html>
```

**실습 목표:**
- Named slot 활용
- 복잡한 레이아웃 구성
- 슬롯별 스타일링

### 예제 4-4: Slot 이벤트와 동적 변경

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>동적 Slot</title>
</head>
<body>
    <dynamic-list>
        <li slot="item">항목 1</li>
        <li slot="item">항목 2</li>
        <li slot="item">항목 3</li>
    </dynamic-list>

    <button id="add">항목 추가</button>
    <button id="remove">마지막 항목 제거</button>

    <script>
        class DynamicList extends HTMLElement {
            constructor() {
                super();
                this.attachShadow({ mode: 'open' });

                this.shadowRoot.innerHTML = `
                    <style>
                        .list {
                            border: 2px solid #3498db;
                            border-radius: 8px;
                            padding: 20px;
                            margin: 20px 0;
                        }

                        h3 {
                            margin-top: 0;
                            color: #3498db;
                        }

                        ::slotted(li) {
                            padding: 10px;
                            margin: 5px 0;
                            background: #ecf0f1;
                            border-radius: 5px;
                            list-style: none;
                        }

                        .count {
                            color: #7f8c8d;
                            font-size: 14px;
                        }
                    </style>

                    <div class="list">
                        <h3>동적 리스트</h3>
                        <p class="count">항목 수: <span id="count">0</span></p>
                        <ul>
                            <slot name="item"></slot>
                        </ul>
                    </div>
                `;

                // slotchange 이벤트 리스닝
                const slot = this.shadowRoot.querySelector('slot[name="item"]');
                slot.addEventListener('slotchange', () => {
                    this.updateCount();
                });
            }

            connectedCallback() {
                this.updateCount();
            }

            updateCount() {
                const slot = this.shadowRoot.querySelector('slot[name="item"]');
                const items = slot.assignedElements();
                const countSpan = this.shadowRoot.getElementById('count');
                countSpan.textContent = items.length;
            }
        }

        customElements.define('dynamic-list', DynamicList);

        // 버튼 이벤트
        let itemCounter = 4;
        const list = document.querySelector('dynamic-list');

        document.getElementById('add').addEventListener('click', () => {
            const li = document.createElement('li');
            li.slot = 'item';
            li.textContent = `항목 ${itemCounter++}`;
            list.appendChild(li);
        });

        document.getElementById('remove').addEventListener('click', () => {
            const items = list.querySelectorAll('[slot="item"]');
            if (items.length > 0) {
                items[items.length - 1].remove();
            }
        });
    </script>
</body>
</html>
```

**실습 목표:**
- `slotchange` 이벤트 활용
- `assignedElements()` 사용법
- 동적 콘텐츠 관리

---

## 5. 실전 프로젝트

### 프로젝트 5-1: 탭 컴포넌트

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>탭 컴포넌트</title>
</head>
<body>
    <tab-container>
        <tab-panel label="소개">
            <h2>Web Components 소개</h2>
            <p>재사용 가능한 커스텀 요소를 만드는 웹 표준 기술입니다.</p>
        </tab-panel>

        <tab-panel label="장점">
            <h2>주요 장점</h2>
            <ul>
                <li>프레임워크 독립적</li>
                <li>캡슐화된 스타일과 로직</li>
                <li>높은 재사용성</li>
            </ul>
        </tab-panel>

        <tab-panel label="사용법">
            <h2>사용 방법</h2>
            <p>Custom Elements, Shadow DOM, Templates를 조합하여 사용합니다.</p>
        </tab-panel>
    </tab-container>

    <script>
        class TabContainer extends HTMLElement {
            constructor() {
                super();
                this.attachShadow({ mode: 'open' });
                this.selectedIndex = 0;
            }

            connectedCallback() {
                this.render();
                this.setupEventListeners();
            }

            render() {
                const panels = Array.from(this.children);

                this.shadowRoot.innerHTML = `
                    <style>
                        .tabs {
                            border-bottom: 2px solid #ddd;
                            margin-bottom: 20px;
                        }

                        .tab-button {
                            background: none;
                            border: none;
                            padding: 15px 30px;
                            font-size: 16px;
                            cursor: pointer;
                            border-bottom: 3px solid transparent;
                            transition: all 0.3s ease;
                        }

                        .tab-button:hover {
                            background: #f5f5f5;
                        }

                        .tab-button.active {
                            color: #667eea;
                            border-bottom-color: #667eea;
                        }

                        .content {
                            padding: 20px;
                        }
                    </style>

                    <div class="tabs">
                        ${panels.map((panel, index) => `
                            <button class="tab-button ${index === this.selectedIndex ? 'active' : ''}"
                                    data-index="${index}">
                                ${panel.getAttribute('label')}
                            </button>
                        `).join('')}
                    </div>

                    <div class="content">
                        <slot></slot>
                    </div>
                `;

                // 패널 표시/숨김
                panels.forEach((panel, index) => {
                    panel.style.display = index === this.selectedIndex ? 'block' : 'none';
                });
            }

            setupEventListeners() {
                this.shadowRoot.querySelectorAll('.tab-button').forEach(button => {
                    button.addEventListener('click', (e) => {
                        this.selectedIndex = parseInt(e.target.dataset.index);
                        this.render();
                        this.setupEventListeners();
                    });
                });
            }
        }

        class TabPanel extends HTMLElement {
            constructor() {
                super();
            }
        }

        customElements.define('tab-container', TabContainer);
        customElements.define('tab-panel', TabPanel);
    </script>
</body>
</html>
```

### 프로젝트 5-2: 모달 다이얼로그

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>모달 다이얼로그</title>
</head>
<body>
    <button id="open">모달 열기</button>

    <modal-dialog id="myModal">
        <h2 slot="header">알림</h2>
        <div slot="body">
            <p>Web Components로 만든 모달입니다!</p>
            <p>Shadow DOM으로 스타일이 완벽히 격리됩니다.</p>
        </div>
        <button slot="footer" class="confirm">확인</button>
        <button slot="footer" class="cancel">취소</button>
    </modal-dialog>

    <script>
        class ModalDialog extends HTMLElement {
            constructor() {
                super();
                this.attachShadow({ mode: 'open' });
            }

            connectedCallback() {
                this.shadowRoot.innerHTML = `
                    <style>
                        .overlay {
                            display: none;
                            position: fixed;
                            top: 0;
                            left: 0;
                            width: 100%;
                            height: 100%;
                            background: rgba(0, 0, 0, 0.5);
                            z-index: 1000;
                            animation: fadeIn 0.3s ease;
                        }

                        .overlay.open {
                            display: flex;
                            align-items: center;
                            justify-content: center;
                        }

                        .modal {
                            background: white;
                            border-radius: 12px;
                            box-shadow: 0 10px 40px rgba(0,0,0,0.3);
                            max-width: 500px;
                            width: 90%;
                            animation: slideUp 0.3s ease;
                        }

                        .header {
                            padding: 20px 30px;
                            border-bottom: 1px solid #eee;
                        }

                        ::slotted([slot="header"]) {
                            margin: 0;
                            color: #333;
                        }

                        .body {
                            padding: 30px;
                            color: #666;
                        }

                        .footer {
                            padding: 20px 30px;
                            border-top: 1px solid #eee;
                            display: flex;
                            justify-content: flex-end;
                            gap: 10px;
                        }

                        ::slotted([slot="footer"]) {
                            padding: 10px 20px;
                            border: none;
                            border-radius: 5px;
                            cursor: pointer;
                            transition: all 0.3s;
                        }

                        ::slotted(.confirm) {
                            background: #667eea;
                            color: white;
                        }

                        ::slotted(.cancel) {
                            background: #e0e0e0;
                            color: #666;
                        }

                        @keyframes fadeIn {
                            from { opacity: 0; }
                            to { opacity: 1; }
                        }

                        @keyframes slideUp {
                            from {
                                transform: translateY(50px);
                                opacity: 0;
                            }
                            to {
                                transform: translateY(0);
                                opacity: 1;
                            }
                        }
                    </style>

                    <div class="overlay" id="overlay">
                        <div class="modal">
                            <div class="header">
                                <slot name="header">제목</slot>
                            </div>
                            <div class="body">
                                <slot name="body">내용</slot>
                            </div>
                            <div class="footer">
                                <slot name="footer"></slot>
                            </div>
                        </div>
                    </div>
                `;

                // 이벤트 리스너
                this.shadowRoot.getElementById('overlay').addEventListener('click', (e) => {
                    if (e.target.id === 'overlay') {
                        this.close();
                    }
                });

                // 슬롯 버튼 이벤트
                const footerSlot = this.shadowRoot.querySelector('slot[name="footer"]');
                footerSlot.addEventListener('click', (e) => {
                    if (e.target.classList.contains('confirm')) {
                        console.log('확인 클릭');
                        this.dispatchEvent(new CustomEvent('confirm'));
                        this.close();
                    } else if (e.target.classList.contains('cancel')) {
                        console.log('취소 클릭');
                        this.dispatchEvent(new CustomEvent('cancel'));
                        this.close();
                    }
                });
            }

            open() {
                this.shadowRoot.getElementById('overlay').classList.add('open');
                this.dispatchEvent(new CustomEvent('open'));
            }

            close() {
                this.shadowRoot.getElementById('overlay').classList.remove('open');
                this.dispatchEvent(new CustomEvent('close'));
            }
        }

        customElements.define('modal-dialog', ModalDialog);

        // 사용 예제
        const modal = document.getElementById('myModal');
        document.getElementById('open').addEventListener('click', () => {
            modal.open();
        });

        modal.addEventListener('confirm', () => {
            console.log('모달 확인됨');
        });

        modal.addEventListener('cancel', () => {
            console.log('모달 취소됨');
        });
    </script>
</body>
</html>
```

### 프로젝트 5-3: 드롭다운 메뉴

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>드롭다운 메뉴</title>
</head>
<body>
    <dropdown-menu label="메뉴 선택">
        <dropdown-item value="option1">옵션 1</dropdown-item>
        <dropdown-item value="option2">옵션 2</dropdown-item>
        <dropdown-item value="option3">옵션 3</dropdown-item>
        <dropdown-item value="option4">옵션 4</dropdown-item>
    </dropdown-menu>

    <p>선택된 값: <span id="selected">없음</span></p>

    <script>
        class DropdownMenu extends HTMLElement {
            constructor() {
                super();
                this.attachShadow({ mode: 'open' });
                this.isOpen = false;
            }

            connectedCallback() {
                this.render();
                this.setupEventListeners();
            }

            render() {
                const label = this.getAttribute('label') || '선택하세요';

                this.shadowRoot.innerHTML = `
                    <style>
                        :host {
                            display: inline-block;
                            position: relative;
                            width: 200px;
                        }

                        .select-button {
                            width: 100%;
                            padding: 12px 16px;
                            background: white;
                            border: 2px solid #ddd;
                            border-radius: 8px;
                            cursor: pointer;
                            display: flex;
                            justify-content: space-between;
                            align-items: center;
                            transition: all 0.3s;
                        }

                        .select-button:hover {
                            border-color: #667eea;
                        }

                        .select-button.open {
                            border-color: #667eea;
                            border-bottom-left-radius: 0;
                            border-bottom-right-radius: 0;
                        }

                        .arrow {
                            transition: transform 0.3s;
                        }

                        .arrow.open {
                            transform: rotate(180deg);
                        }

                        .dropdown {
                            position: absolute;
                            top: 100%;
                            left: 0;
                            right: 0;
                            background: white;
                            border: 2px solid #667eea;
                            border-top: none;
                            border-bottom-left-radius: 8px;
                            border-bottom-right-radius: 8px;
                            max-height: 0;
                            overflow: hidden;
                            transition: max-height 0.3s ease;
                            z-index: 1000;
                        }

                        .dropdown.open {
                            max-height: 300px;
                            overflow-y: auto;
                        }

                        ::slotted(dropdown-item) {
                            display: block;
                            padding: 12px 16px;
                            cursor: pointer;
                            transition: background 0.2s;
                        }

                        ::slotted(dropdown-item:hover) {
                            background: #f0f0f0;
                        }
                    </style>

                    <div class="select-button" id="button">
                        <span id="label">${label}</span>
                        <span class="arrow">▼</span>
                    </div>

                    <div class="dropdown" id="dropdown">
                        <slot></slot>
                    </div>
                `;
            }

            setupEventListeners() {
                const button = this.shadowRoot.getElementById('button');
                const dropdown = this.shadowRoot.getElementById('dropdown');
                const arrow = this.shadowRoot.querySelector('.arrow');

                button.addEventListener('click', () => {
                    this.isOpen = !this.isOpen;
                    dropdown.classList.toggle('open', this.isOpen);
                    button.classList.toggle('open', this.isOpen);
                    arrow.classList.toggle('open', this.isOpen);
                });

                // 외부 클릭 시 닫기
                document.addEventListener('click', (e) => {
                    if (!this.contains(e.target)) {
                        this.isOpen = false;
                        dropdown.classList.remove('open');
                        button.classList.remove('open');
                        arrow.classList.remove('open');
                    }
                });

                // 아이템 클릭 이벤트
                this.addEventListener('item-selected', (e) => {
                    this.shadowRoot.getElementById('label').textContent = e.detail.text;
                    this.isOpen = false;
                    dropdown.classList.remove('open');
                    button.classList.remove('open');
                    arrow.classList.remove('open');
                });
            }
        }

        class DropdownItem extends HTMLElement {
            connectedCallback() {
                this.addEventListener('click', () => {
                    const value = this.getAttribute('value');
                    const text = this.textContent;

                    this.dispatchEvent(new CustomEvent('item-selected', {
                        bubbles: true,
                        composed: true,
                        detail: { value, text }
                    }));

                    // 외부로 이벤트 전파
                    this.closest('dropdown-menu').dispatchEvent(
                        new CustomEvent('change', {
                            detail: { value, text }
                        })
                    );
                });
            }
        }

        customElements.define('dropdown-menu', DropdownMenu);
        customElements.define('dropdown-item', DropdownItem);

        // 사용 예제
        document.querySelector('dropdown-menu').addEventListener('change', (e) => {
            document.getElementById('selected').textContent =
                `${e.detail.text} (${e.detail.value})`;
        });
    </script>
</body>
</html>
```

**실습 목표:**
- 복잡한 상호작용 구현
- 이벤트 버블링과 `composed` 속성
- 외부 클릭 감지 패턴

---

## 추가 학습 리소스

### 공식 문서
- [MDN Web Components](https://developer.mozilla.org/ko/docs/Web/Web_Components)
- [Custom Elements v1](https://html.spec.whatwg.org/multipage/custom-elements.html)
- [Shadow DOM v1](https://dom.spec.whatwg.org/#shadow-trees)

### 고급 주제
1. **Form Associated Custom Elements** - 폼 연동
2. **Constructible Stylesheets** - 동적 스타일시트
3. **CSS Shadow Parts** - Shadow DOM 스타일 외부 제어
4. **Declarative Shadow DOM** - HTML로 Shadow DOM 선언

### 실전 팁
- 브라우저 호환성 확인 (polyfill 사용)
- 성능 최적화 (lazy loading, 이벤트 위임)
- 접근성 고려 (ARIA 속성)
- 타입스크립트 통합

---

## 학습 순서 요약

1. ✅ **1주차**: Custom Elements 기초 (예제 1-1 ~ 2-2)
2. ✅ **2주차**: Shadow DOM 이해 (예제 3-1 ~ 3-3)
3. ✅ **3주차**: Templates & Slots (예제 4-1 ~ 4-4)
4. ✅ **4주차**: 실전 프로젝트 (예제 5-1 ~ 5-3)

각 예제를 순서대로 따라하면서 콘솔 로그를 확인하고, 코드를 수정해보며 학습하세요!
