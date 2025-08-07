---
title: "자료구조 Trie"
datePublished: Thu Aug 07 2025 07:17:45 GMT+0000 (Coordinated Universal Time)
cuid: cme12eibo003x02kzfjzf5fst
slug: trie
tags: java, trie, datastructure

---

## Trie란?

Trie는 문자열 탐색에 특화된 트리 기반의 자료구조로, Prefix Tree(접두사 트리) 라고도 불립니다.

문자열의 접두사(prefix)를 기준으로 구조화되어 있기 때문에, 자동완성, 사전 검색, 검색 최적화 등에서 널리 활용됩니다.

**아래의 Trie에 들어있는 문자열**

apple, april, bus, busy, beer, best

![](https://blog.kakaocdn.net/dna/n3DKG/btsjYyjBfNc/AAAAAAAAAAAAAAAAAAAAAIRd_1nkUDoptePSiUaseNFIq3jZP_Z6ahf-CAr6r6rP/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1756652399&allow_ip=&allow_referer=&signature=KhnrRDoHVTB6ejK8h%2B9VrDHkmOs%3D align="left")

## 주요 특징

* 각 노드는 문자열의 한 글자를 저장
    
* 루트 노드는 빈 문자열을 나타냄
    
* 같은 접두사를 공유하는 단어들끼리 같은 경로를 따라감 → 공간 효율이 높음
    
* 노드마다 ‘isEndOfWord’ 플래그를 통해 단어의 끝인지 여부를 판단 (파란 노드)
    

## 노드 정의

```java
public class TrieNode {

    private HashMap<Character, TrieNode> children = new HashMap<>();
    //현재 노드에서 이어질 수 있는 문자들과 그에 대응하는 하위 TrieNode들을 저장하는 map
    
    private boolean isEndOfWord = false;  // 어떤 단어의 마지막 글자인지를 나타내는 플래그

    public HashMap<Character, TrieNode> getChildren() {
        return children;
    }

    public boolean isEndOfWord() {
        return isEndOfWord;
    }

    public void setEndOfWord(boolean isEndOfWord) {
        this.isEndOfWord = isEndOfWord;
    }
}
```

---

## 기본 연산 구현

### 1\. 삽입 (Insert)

```java

public void insert(String word) {
    TrieNode current = root;
    for (char ch : word.toCharArray()) {
        current = current.getChildren().computeIfAbsent(ch, c -> new TrieNode());
    }
    current.setEndOfWord(true);
}

```

* 한 글자씩 순회하며 존재하지 않으면 새 노드를 생성
    
* 마지막 글자 노드에는 isEndOfWord를 true로 설정
    
* 시간 복잡도 : O(n) (n= 문자열 길이)
    

---

### 2\. 탐색 (Search)

```java
public boolean search(String word){
	TrieNode current = root;
	for(char ch : word.toCharArray()){
		TrieNode node = current.getChildren().get(ch);
		if(node == null) return false;
		current = node;
	}
	return current.isEndOfWord();
}
```

* 한 글자씩 트리를 따라가며 , 노드가 없으면 false
    
* isEndOfWord가 true이면 해당 단어가 존재
    

---

### 3\. 삭제 (Delete)

```java
// Trie에서 문자열을 삭제하는 메서드 (사용자 호출용)
public void delete(String word) {
    delete(root, word, 0);
}

/**
 * Trie에서 재귀적으로 문자열을 삭제하는 내부 메서드
 *
 * @param current 현재 탐색 중인 노드
 * @param word 삭제하려는 문자열
 * @param index 현재 탐색 중인 문자 인덱스
 * @return 현재 노드를 삭제할 수 있는지 여부 (true면 상위 노드에서 이 노드를 삭제해도 됨)
 */
private boolean delete(TrieNode current, String word, int index) {
    // 문자열의 끝까지 도달한 경우
    if (index == word.length()) {
        // 현재 노드가 단어의 끝이 아니라면, Trie에 존재하지 않는 단어 → 삭제 실패
        if (!current.isEndOfWord()) {
            return false;
        }

        // 단어의 끝 표시를 제거
        current.setEndOfWord(false);

        // 자식 노드가 없다면 상위 노드에서 이 노드를 삭제할 수 있도록 true 반환
        return current.getChildren().isEmpty();
    }

    // 현재 문자 하나 가져오기
    char ch = word.charAt(index);

    // 해당 문자가 자식 노드에 없다면 → 존재하지 않는 단어 → 삭제 불가
    TrieNode node = current.getChildren().get(ch);
    if (node == null) {
        return false;
    }

    // 자식 노드에서 삭제 재귀 수행
    // 1. 자식 노드에서 단어 삭제가 완료되었고
    // 2. 그 자식 노드가 더 이상 단어의 끝이 아니면
    // → 현재 노드에서 해당 자식 노드를 삭제할 수 있음
    boolean shouldDeleteCurrentNode =
            delete(node, word, index + 1) && !node.isEndOfWord();

    // 자식 노드를 삭제해도 된다면, 현재 노드의 자식 목록에서 해당 문자 제거
    if (shouldDeleteCurrentNode) {
        current.getChildren().remove(ch);

        // 현재 노드도 더 이상 자식이 없다면 true 반환 → 상위 노드에서도 삭제 가능
        return current.getChildren().isEmpty();
    }

    // 삭제할 수 없는 경우 (예: 해당 노드가 다른 단어의 일부일 경우)
    return false;
}

```

참고 자료 : [https://www.baeldung.com/trie-java,https://innovation123.tistory.com/116](https://www.baeldung.com/trie-java,https://innovation123.tistory.com/116)