---
layout: post
title: "정렬 시간복잡도"
categories: [Algorithm]
tags: []
last_modified_at: 2024-01-08
---

시간 복잡도 한 번에 보기 (설명은 후술)

시간 빠르기는 다음과 같음

o(1) > o(logn) > o(n) > o(nlogn) > o(n^2) > o(2^n) > o(n!)

| sorting       | best      | average   | worst     |              |
|---------------|-----------|-----------|-----------|--------------|
| 버블 정렬    | o(n^2)   | o(n^2)   | o(n^2)   |              |
| 선택 정렬    | o(n^2)   | o(n^2)   | o(n^2)   |              |
| 퀵 정렬      | o(nlogn) | o(nlogn) | o(n^2)   |              |
| 힙 정렬      | o(nlogn) | o(nlogn) | o(nlogn) |              |
| 병합 정렬    | o(nlogn) | o(nlogn) | o(nlogn) |              |
| 삽입 정렬    | o(n)     | o(n^2)   | o(n^2)   |              |
| 쉘 정렬      | o(n)     |           | o(n^2)   |              |
| 기수 정렬    | o(n)     | o(n)     | o(n)     |              |
| 카운팅 정렬  | o(n)     | o(n)     | o(n)     |              |