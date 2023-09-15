---
tags:
  - JVM
---
## 목차

1. [개요](#개요)
2. [2.1_The_class_File_Format](#The_class_File_Format)
3. [2.2_Data_Types](#2.2_Data_Types)
4. [2.3_Primitive_Types_and_Values](#2.3_Primitive_Types_and_Values)
5. [2.3.1_Integral_Types_and_Values](#2.3.1_Integral_Types_and_Values)
6. [2.3.2_Floating-Point_Types and_Values](#2.3.2_Floating-Point_Types and_Values)
7. [2.3.3_The_returnAddress_Type_and_Values](#2.3.3_The_returnAddress_Type_and_Values)


# 개요

이 문서에서는 추상적인 시스템에 대해 서술합니다. Java 가상 시스템의 특정 구현에 대해서는 설명하지 않습니다.
Java Virtural Machine을 올바르게 구현하려면 클래스 파일 형식을 읽고 클래스에 지정된 작업을 올바르게 수행하기만 하면 됩니다.
JVM의 사양에 포함되지 않은 구현 세부 정보는 불필요하게 구현자의 창의성을 제약합니다. 예를 들어 런타임 데이터 영역의 `메모리 레이아웃, 사용된 가바지 컬렉션 알고리즘, JVM 명령어의 내부 최적화(기계 코드로 변환)`는 구현자의 재량에 맡깁니다.

본 명세서의 유니코드에 대한 모든 참조는 [unicode](https://www.unicode.org/)에서 구할 수 있는 유니코드 표준 버전 13.0과 관련되어 있습니다.

# 2.1_The_class_File_Format

JVM이 실행할 컴파일된 코드는 하드웨어 및 운영 체제에 독립적인 이진 형식을 사용하여 표현하며, 일반적으로 클래스 파일 형식으로 알려져 있지만 반드시 그렇지는 않습니다. 클래스 파일 형식은 플랫폼별 개체 파일 형식에서 당연하게 여겨질 수 있는 바이트 순서와 같은 세부 정보를 포함하여 클래스 또는 인터페이스의 표현을 정확하게 정의합니다.
`Chapter 4 "The class File Format"` 에서는 클래스 파일 형식에 대해 자세히 설명합니다.


# 2.2_Data_Types

JVM은 자바 프로그래밍 언어와 마찬가지로 `primitive types`과 `reference types` 두 가지로 동작합니다. 따라서 변수에 저장되고 인수로 전달되며 메서에서 반환되거나 조작되는 값에는 기본형과 참조형 두 가지 종류가 있습니다.

일반적으로 JVM의 모든 유형 검사는 대부분 컴파일러에의해 런 타임 전에 수행되므로 JVM 자체에서 수행할 필요가 없습니다.기본 타입의 값은 런 타임에 타입을 결정 혹은 참조 타입의 값과 구별하기 위해 태그를 달거나 다른 방식으로 조사할 필요가 없습니다.
대신, JVM의 명령어 셋은 특정 타입의 값을 처리하기 위해 설계된 명령어를 사용하여 피연산자 타입을 구분합니다. 예를 들어 `iadd, ladd, fadd 및 dadd`는 모두 두 개의 숫자 값이 더해지고 숫자 결과가 생성되는 JVM  명령어이지만 각각은 int, long, float, double의 피연산자 유형에 특화되어 있습니다. JVM 명령어 셋에서 타입 지원 요약에 대한 자세한 내용은 `§2.11.1`을 참조하십시오.

JVM은 개체에 대한 명시적 지원을 포함합니다. 개체는 동적으로 할당된 클래스 인스턴스 또는 배열로 개체에 대한 참조는 JVM 유형 참조를 갖는 것으로 간주됩니다. 유형 참조의 값은 개체에 대한 포인터로 간주될 수 있고, 개체에 대한 참조가 둘 이상 존재할 수 있습니다.
개체는 항상 유형 참조의 값을 통해 동작되고, 전달되고, 테스트됩니다.




# 2.3_Primitive_Types_and_Values

JVM에서 지원하는 기본 데이터 유형은 `numeric types, boolean type (§2.3.4), returnAddress type (§2.3.3)`입니다.
숫자 유형은 `integral types (§2.3.1), floating-point types (§2.3.2)`으로 구성됩니다.

**integral types은 다음과 같습니다.**

- 바이트(byte) : 값은 8비트 부호가 있는 2의 정수이고 기본값은 0입니다.
- short : 값은 16비트 부호의 2의 정수이고 기본값은 0입니다.
- int : 값은 32비트 부호가 있는 2의 정수이고 기본값은 0입니다.
- long : 값은 64비트 부호가 있는 2의 정수이고 기본값은 0입니다.
- char : 기본 다국어 평면에서 유니코드 코드 포인트를 나타내는 16비트 비부호 정수를 UTF-16으로 인코딩하고 기본값은 null 코드 포인트('\u0000')입니다.

**floating-point types은 다음과 같습니다.**

- float : 값이 32비트 IEEE 754 바이너리 32형식으로 표시되는 값과 정확히 일치하고 기본값은 양의 0 입니다.
- double : 값이 64비트 IEEE 754 바이너리 64형식의 값과 정확히 일치하고 기본값은 양의 0 입니다.

`boolean` 유형의 값은 true 값과 false를 인코딩하며 기본값은 false입니다.
Java® Virtual Machine Specification의 First Edition에서는 `boolean을` JVM 유형으로 간주하지 않았습니다. 하지만 JVM에서는 `boolean` 값이 제한적으로 지원됩니다. Java® Virtual Machine Specification의 Second Edition에서는 `boolean을` 유형으로 취급하여 문제를 명확히 했습니다.`returnAddress`유형의 값은 JVM 명령의 `opcode`에 대한 포인터입니다. 기본 유형 중 `returnAddress` 유형만 Java 프로그래밍 언어 유형과 직접 연결되지 않습니다.


# 2.3.1_Integral_Types_and_Values

JVM의 integral types 유형 값은 다음과 같습니다.

- byte :  -128 to 127 (-$2^7$ to $2^7$ - 1), 포함
- short : -32768 to 32767 (-$2^{15}$ to $2^{15}$ - 1), 포함
- int : -2147483648 to 2147483647 ($-2^{31}$ to $2^{31}$ - 1), 포함
- long : - -9223372036854775808 to 9223372036854775807 ($-2^{63}$ to $2^{63}- 1$), 포함
- char :  0 ~ 65535, 포함

# 2.3.2_Floating-Point_Types and_Values

floating-point type은 IEEE 754 표준(JLS § 1.7)에 명시된 대로 IEEE 754 값 및 연산을 위한 32-bit binary32 및 64-bit binary64 floating-point formats와 개념적으로 연관되어 있습니다.

Java SE 15 이상에서 JVM은 IEEE 754 Standard의 2019년 버전을 사용합니다. Java SE 15 이전에는 JVM이 IEEE 754 Standart의 1985년 버전을 사용했는데, 이때 binary32 형식은 단일 형식으로, binary64 형식은 이중 형식으로 알려졌습니다.

IEEE 754는 부호와 크기로 이루어진 양수와 음수 숫자 뿐만 아니라 양수와 음수 제로(zero), 양수와 음수 무한(infinity), 그리고 특수한 NaN(Not-a-Number) 값도 포함하고 있습니다. NaN 값은 0을 0으로 나누는 등의 특정한 유효하지 않은 연산의 결과를 나타내기 위해 사용됩니다. 부동 소수점(float)과 배정밀도(double)의 양식으로 정의된 NaN 상수는 Float.NaN과 Double.NaN으로 미리 정의되어 있습니다.

부동 소수점 타입의 유한한, 영이 아닌 값들은 다음 형식으로 모두 표현할 수 있습니다: s ⋅ m ⋅ 2^(e - N + 1) 여기서

- s는 +1 또는 -1입니다.
- m은 $2^N$ 미만인 양의 정수입니다.
- e는 Emin = $-(2^{K-1}-2)$부터 Emax = $2^{K-1}-1$까지 (포함)의 정수이며,
- N과 K는 해당 타입에 따라 달라지는 매개변수입니다.

일부 값들은 이 형식으로 여러 가지 방식으로 표현될 수 있습니다. 예를 들어, 어떤 부동 소수점 타입의 값 v가 특정한 s, m 및 e 값들을 사용하여 이 형식으로 표현될 수 있다고 가정해보겠습니다. 그리고 만약 m이 짝수이고 e가 $2^{K-1}$보다 작다면, m을 반으로 줄이고 e를 1 증가시켜 동일한 값 v에 대한 두 번째 표현을 생성할 수 있습니다.

이 형식으로 표현된 값이 m ≥ $2^{N-1}$인 경우, 정규화(normalized)된 것이라고 합니다. 그렇지 않은 경우, 표현은 아래수(subnormal)라고 합니다. 부동 소수점 타입의 값이 m ≥ $2^{N-1}$를 만족하는 방식으로 표현할 수 없다면, 해당 값은 아래수(subnormal) 값이라고 합니다. 이는 해당 값의 크기가 가장 작은 정규화된 값의 크기 미만이기 때문입니다.

매개변수 N과 K (그리고 파생된 매개변수 Emin과 Emax)에 대한 제약사항은 float와 double에 대한 Table 2.3.2-A에 요약되어 있습니다.

# 2.3.2-A. Floating-point parameters 표


![](https://i.imgur.com/5Ak5OfU.png)

NaN을 제외한 부동 소수점 값은 순서가 있습니다. 가장 작은 값부터 가장 큰 값까지 나열하면, 음의 무한대, 음의 유한한 영이 아닌 값, 양수와 음수 제로, 양의 유한한 영이 아닌 값, 그리고 양의 무한대입니다.

IEEE 754는 각각의 binary32와 binary64 부동 소수점 형식에 대해 여러 가지 다른 NaN 값을 허용합니다. 그러나 Java SE 플랫폼은 일반적으로 주어진 부동 소수점 형식의 NaN 값을 하나의 대표적인 값으로 취급하며, 이에 따라 이 명세서는 보통 임의의 NaN을 대표적인 값으로 언급합니다.

IEEE 754에 따라 NaN이 아닌 인자를 가지고 수행되는 부동 소수점 연산은 NaN 결과를 생성할 수 있습니다. IEEE 754는 일련의 NaN 비트 패턴을 지정하지만 NaN 결과를 나타내기 위해 어떤 특정한 NaN 비트 패턴을 사용해야 하는지를 강제하지 않으며, 이 부분은 하드웨어 아키텍처에 맡겨집니다. 프로그래머는 다른 비트 패턴을 가진 NaN을 생성하여 후행 진단 정보 등을 인코딩할 수 있습니다. 이러한 NaN 값은 각각 float 및 double에 대해 Float.intBitsToFloat 및 Double.longBitsToDouble 메서드를 사용하여 생성할 수 있습니다. 또한, NaN 값의 비트 패턴을 검사하기 위해서는 Float.floatToRawIntBits 및 Double.doubleToRawLongBits 메서드를 각각 float와 double에 대해 사용할 수 있습니다.

양의 제로와 음의 제로는 동등하게 비교됩니다. 그러나 그들을 구별할 수 있는 다른 연산도 있습니다. 예를 들어, 1.0을 0.0으로 나누면 양의 무한대가 생성되지만, 1.0을 -0.0으로 나누면 음의 무한대가 생성됩니다.

NaN은 순서가 없으므로 숫자 비교 및 숫자 동등성 테스트는 그들의 피연산자 중 하나 이상이 NaN인 경우 값이 false입니다. 특히, 어떤 값에 대한 숫자 동등성 테스트는 값이 NaN이면 값은 false이며, 숫자 불일치 테스트는 어떤 피연산자가 NaN인 경우 값이 true입니다.

# 2.3.3_The_returnAddress_Type_and_Values
