---
sticker: lucide//user-check
tags: JVM
Bit-of-History: ""
---
## 목차

1. [A Bit of History](#A Bit of History)
2. [Organization of the Specification](#Organization of the Specification)
3. [Notation](#Notation)
4. [Feedback](#Feedback)


# A Bit of History

Java 프로그래밍 언어는 범용, 동시성, 객체 지향을 지원하는 언어로 그 문법은 C와 C++과 유사하지만
C 및 C++을 복잡하고 혼란스럽게 만드는 많은 기능을 제거하고 안전성을 갖추고 있다.

자바 플랫폼은 네트워크 연결된 networked consumer devices에 소프트웨어를 개발하는 문제를 해결하기 위해 개발되었는데 이를 통해 여러 호스트 아키텍처를 지원하고 소프트웨어 구성 요소를 안전하게 전달할 수 있도록 설계되었다. 

초기 목적을 충족하기 위해 컴파일된 코드는 네트워크를 통해 전송되어야하며 모든 클라이언트에서 작동하고 클라이언트에게 실행이 안전함을 보장해야 했다.