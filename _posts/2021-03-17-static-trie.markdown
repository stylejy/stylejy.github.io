---
layout: post
title: entity-state
date: 2021-03-17
# img: jpa-mapping.jpg # Add image post (optional)
tags: [Blog, Study, cpp, Trie]
author: Seokkyu Ryu # Add name author (required-fixed)
author-pic: sk.jpg # Add author-pic (required-fixed)
about-author: screw.. # Add about-author (about 과 같음) (optional-fixed)
email: tjrrb95@gmail.com # email (optiona-fixed)
---

```cpp
#include <cstdio>

const int MAXN = 500; //전체 들어오는 최대 문자열의 갯수가 50개라고 칠때,
int n, pos, i, j, k, find, x, res, idx, ccnt;
char str[10]; // 트라이에 저장할 문자열을 저장 
int tmp; // 사전순대로 트라이에 탐색했을 때, 그 과정에서 문자열의 해시값(?)을 임시적으로 저장. 
char sample[6]; //검색할 단어 
int nex[MAXN * 8][26]; // 다음 노드 번호
int cc[MAXN * 8]; // 이 노드 번호로 끝나는 단어의 갯수 
char ch[MAXN * 8]; // 해당 노드 번호의 문자
//최종배열은 인덱스가 앞에서부터 높은 우선순위로 정함 내림차순으로 설정
int ans_cnt[5]; //최종배열을 5개씩 유지하며 cnt값을 저장 
int ans_str[5]; //최종배열을 5개씩 유지하며 문자열의 해싱값 저장

void init() {
	res = 0; find = 0; tmp = 0;
	for (i = 0; i < 5; i++) {
		ans_cnt[i] = 0;
		ans_str[i] = 0;
	}
}

void add(int now, int state) {
	if (str[state] == '\0') {
		cc[now]++; // 중복된 문자열 개수( 기존 finish배열 + 같은 문자 갯수)
		idx++;
		return;
	}
	int cur = str[state] - 'A';
	if (!nex[now][cur]) nex[now][cur] = pos++;
	int nn = nex[now][cur];
	ch[nn] = str[state];
	add(nn, state + 1);
}

void dfs(int now, int _a, int pos) { // now부터 사전순 탐색 부분
	int _k;
	if (cc[now]) {  //문자열이 끝났으면
		find++; // 찾은 갯수 ++
		ccnt = find > 5 ? 5 : find;
		for (_k = 0; _k < 5; _k++) {
			if (cc[now] > ans_cnt[_k]) {
				int kk;
				for (kk = ccnt - 1; kk > _k; kk--) {
					ans_cnt[kk] = ans_cnt[kk - 1];
					ans_str[kk] = ans_str[kk - 1];
				}
				// 여기서부터 _a를 해싱해야한다.
				ans_cnt[_k] = cc[now]; // cnt값을 저장 문자열의 갯수 저장
				ans_str[_k] = _a; // 문자열의 해싱값 저장
				break;
			}
		}
	}
	for (_k = 0; _k < 26; _k++) { // 자기 자식노드 26개중 0ㅇ이아닌 값이 있으면 거기로 dfs ㄱㄱ
		int nn = nex[now][_k];
		if (nn) {
			dfs(nn, _a + (_k+1) *pos , pos*27); // pos는 곱해줄 자릿값
		}
	}
}

int search(int now, int state) { //ABCD로 TRIE 탐색 후 없으면 -1 리턴하고 abcd까지 다 있으면 그 노드 번호 리턴
	if (state == 4) {
		return now;
	}
	int cur = str[state] - 'A';
	int nn = nex[now][cur];
	if ((sample[state] != ch[nn]) && (state < 4)) {
		return -1;
	}
	search(nn, state + 1);
}
void print() {
	printf("%d\n", ccnt);
	for (k = 0; k < ccnt; k++) {
		char r[5] = { '\0', };
		for (j = 0; j < 4; j++) {
			int state = (ans_str[k] % 27);
			if (!state) break;
			r[j] = state - 1 + 'A';
			ans_str[k] /= 27;
		}
		printf("%d %s%s", ans_cnt[k], sample, r);
		puts("");
	}
}
int main() {
	freopen("input.txt", "r", stdin);
	scanf("%d", &n);
	idx = 0;
	pos = 1;
	while (n--) {
		init();
		scanf("%d", &x);
		if (x == 1) {
			scanf("%s", str);
			add(0, 0);
		}
		else {
			scanf("%s", sample);
			int root = search(0, 0); //노드번호 리턴 받는다
			if (root == -1) continue;
			dfs(root, tmp, 1); // 그 노드번호부터 트라이 사전순으로 탐색
			print();
		}
	}
	return 0;
}
```

