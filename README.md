# BMP 이미지 변환 소프트웨어

![output_grayscale_map.bmp](https://github.com/user-attachments/files/22039908/output_grayscale_map.bmp)

## 문서 관리 정보
 - 문서 ID: BMP-DO178C-001
 - 버전: 1.2 / 날짜: 2025년 8월 29일
 - 분류: 설계 보증 레벨 D (DAL-D)
 - 시스템: BMP 이미지 파일 활용

### 1. 소프트웨어 인증 계획서 (PSAC)
#### 1.1 소프트웨어 개요
 - 소프트웨어 항목: BMP 그레이스케일 변환기 기능: 24비트 컬러 BMP 이미지를 8비트그레이스케일로 변환 중요도 레벨: DAL-D (경미한 고장 상황)

#### 1.2 소프트웨어 생명주기 프로세스
 - 계획 수립 프로세스
 - 소프트웨어 개발 프로세스
 - 소프트웨어 검증 프로세스
 - 소프트웨어 형상관리 프로세스
 - 소프트웨어 품질보증 프로세스

#### 1.3 소프트웨어 생명주기 데이터

### 2. 소프트웨어 요구사항 표준 (SRS)
#### 2.1 상위레벨 요구사항
 * HLR-001 : 이미지 형식 검증
   * 소프트웨어는 입력 파일이 24비트 BMP 형식임을 검증해야 한다
   * 소프트웨어는 이미지 크기가 630x630 픽셀임을 확인해야 한다
   * 소프트웨어는 유효하지 않은 파일 형식을 적절한 오류 메시지와 함께 거부해야 한다

 * HLR-002 : 색공간 변환
   * 소프트웨어는 표준 휘도 공식을 사용하여 RGB 픽셀을 그레이스케일로 변환해야 한다:
   * $Y = 0.299R + 0.587G + 0.114*B$
   * 변환 과정에서 픽셀의 공간적 관계를 유지해야 한다
   * 출력 값은 0-255 범위 내에 있어야 한다

 * HLR-003 : 출력 파일 생성
   * 소프트웨어는 8비트 그레이스케일 BMP 파일을 생성해야 한다
   * 소프트웨어는 256색 그레이스케일 팔레트를 포함해야 한다
   * 출력 파일은 BMP 표준을 준수해야 한다
 * HLR-004 : 메모리 관리
   * 소프트웨어는 동적 메모리 할당 실패를 감지하고 처리해야 한다
   * 소프트웨어는 할당된 모든 메모리를 적절히 해제해야 한다
   * 메모리 누수가 발생하지 않아야 한다
 * HLR-005 : 오류 처리
   * 소프트웨어는 파일 입출력 오류를 감지하고 보고해야 한다소프트웨어는 비정상 종료 시 리소스를 정리해야 한다
   * 모든 오류 상황에 대해 명확한 메시지를 제공해야 한다

#### 2.2 저수준 요구사항
 * LLR-001 : BMP 헤더 검증
   * 파일 시그니처가 0x4D42 ("BM")인지 확인
   * bitCount 필드가 24인지 검증
   * width와 height가 각각 630인지 확인
 * LLR-002 : 픽셀 데이터 읽기
   * BMP 픽셀 데이터를 하단부터 상단 순으로 읽기
   * 각 행의 패딩 바이트 처리
   * RGB 구조체로 픽셀 데이터 저장
 * LLR-003 : 그레이스케일 변환 알고리즘
   * 각 픽셀에 대해 Y = 0.299
   * R + 0.587
   * G + 0.114*B 공식 적용
   * 결과를 uint8_t 형으로 캐스팅
   * 변환된 값을 별도 배열에 저장
 * LLR-004 : 출력 BMP 생성
   * 8비트 BMP용 헤더 구조 생성
   * 256색 그레이스케일 팔레트 작성
   * 그레이스케일 데이터를 하단부터 상단 순으로 출력

### 3. 소프트웨어 설계 표준 (SDS)
#### 3.1 아키텍처 설계
   * 모듈 구조:
~~~
BMP__Processor
├─── File_Handler
│  ├─── Input_Validator
│  └─── Output_Generator
├─── Image_Processor
│  ├─── Color__Converter
│  └─── Memory__Manager
└─── Error_Handler
~~~

#### 3.2 상세 설계
 * 구조체 정의:
   * BMPFileHeader: BMP 파일 헤더 정보
   * BMPInfoHeader: BMP 정보 헤더
   * RGB: 24비트 RGB 픽셀 데이터
 * 주요 함수:
   * main() : 메인 프로세스 제어
   * rgbToGrayscale() : RGB-그레이스케일 변환
 * 파일 I/O 처리 루틴
 * b메모리 할당/해제 루틴

#### 3.3 인터페이스 설계
 * 입력 인터페이스:
   * 파일명: "map.bmp"
   * 형식: 24비트 컬러 BMP
   * 크기: 4183x4183 픽셀
 * 출력 인터페이스:
   * 파일명: "output_grayscale.bmp"
   * 형식: 8비트 그레이스케일 BMP
   * 팔레트: 256색 그레이스케일4. 소프트웨어 코드 표준 (SCS)

#### 4.1 코딩 규칙
 * 명명 규칙:
   * 구조체: PascalCase (예: BMPFileHeader)
   * 함수: camelCase (예: rgbToGrayscale)
   * 변수: camelCase (예: imageData)
   * 상수: UPPER_CASE (예: IMAGE_WIDTH)
 * 코드 스타일:
   * 들여쓰기: 4칸 스페이스
   * 중괄호: K&R 스타일
   * 한 줄당 하나의 명령문
   * 적절한 주석 작성

#### 4.2 품질 기준
   * 가독성:
   * 명확한 변수명 사용
   * 적절한 주석 포함
   * 일관된 코드 형식
   * 유지보수성:
   * 모듈화된 구조
   * 오류 처리 포함
   * 메모리 관리 규칙 준수

### 5. 소프트웨어 검증 계획 (SVP)
#### 5.1 검증 목표
   * 모든 요구사항이 올바르게 구현되었는지 확인
   * 소프트웨어가 항공 안전 기준을 충족하는지 검증
   * 오류 조건에서의 소프트웨어 동작 검증

#### 5.2 검증 방법
 * 정적 분석:
   * 코드 리뷰
   * MISRA C 규칙 준수 검사
   * 정적 분석 도구 사용
 * 동적 테스트:
   * 단위 테스트
   * 통합 테스트
   * 시스템 테스트

#### 5.3 테스트 케이스
 * TC-001 : 정상 입력 처리
   * 유효한 630x630 24비트 BMP 파일 입력
   * 예상 결과: 성공적인 그레이스케일 변환 및 출력
 * TC-002 : 잘못된 파일 형식
   * 비BMP 파일 입력
   * 예상 결과: 오류 메시지 출력 및 프로그램 종료
 * TC-003 : 잘못된 이미지 크기
   * 630x630이 아닌 BMP 파일 입력
   * 예상 결과: 크기 오류 메시지 출력
 * TC-004 : 메모리 부족 상황
   * 메모리 할당 실패 시뮬레이션예상 결과: 적절한 오류 처리 및 리소스 정리
 * TC-005 : 파일 I/O 오류
   * 읽기/쓰기 권한 없는 파일 처리
   * 예상 결과: I/O 오류 메시지 출력

#### 5.4 커버리지 기준
 * 명령문 커버리지: 100%
 * 분기 커버리지: 100%
 * MC/DC (Modified Condition/Decision Coverage): 해당 없음 (DAL-D)

### 6. 소프트웨어 형상관리 계획 (SCMP)
#### 6.1 형상관리 목표
   * 소프트웨어 생명주기 동안 모든 변경사항 추적
   * 베이스라인 관리 및 버전 제어
   * 변경 요청 및 승인 프로세스 관리

#### 6.2 형상 식별
   * 소스 코드 파일
   * 실행 파일
   * 문서화 자료
   * 테스트 케이스 및 결과

#### 6.3 변경 관리
   * 변경 요청 프로세스:
    1. 변경 요청서 작성
    2. 기술 검토
    3. 영향 분석
    4. 승인/거부 결정
    5. 구현 및 검증
    6. 형상 업데이트

### 7. 추적성 매트릭스

 | 요구사항 ID |  설계 요소 | 코드 구현 | 테스트 케이스 |
 |-----|-----|-----|-----|
 | HLR-001 | File_Validator | fread(), 헤더 검증 | TC-001, TC-002, TC-003 |
 | HLR-002 | Color_Converter | rgbToGrayscale() | TC-001 |
 | HLR-003 | Output_Generator | BMP 생성 루틴 | TC-001 |
 | HLR-004 | Memory_Manager | malloc(), free() | TC-004 |
 | HLR-005 | Error_Handler | 오류 처리 루틴 | TC-002~TC-005 |

### 8. 인증 결론
   * 본 BMP 그레이스케일 변환 소프트웨어는 DO-178C DAL-D 수준의 요구사항을 충족하도록 설계 및 구현되었습 니다.
   * 모든 필수 문서가 작성되었으며, 계획된 검증 활동을 통해 소프트웨어의 안전성과 신뢰성이 입증될 것입니 다.

## 승인:
   * 개발팀장: ________________
   * 품질보증 관리자: ________________
   * 인증 담당자: ________________
날짜:
2025년 8월 28일


# BMP 그레이스케일 이미지 MEM 변환
<br>

verilog simulation을 위해 이미지의 각 픽셀을 16진수로 변환하고 <br>
0x0D 0x0A(ASCII: Carriage Return, Line Feed)로 각 픽셀을 구분한다.

<br>

__예시__

<img width="470" height="423" alt="스크린샷 2025-08-29 115007" src="https://github.com/user-attachments/assets/9e9ad650-ff86-4731-8920-5ce19f227276" />

<br>

### 1. ======== 파일 생성 추가 ========
```
const char* memFile = "output_image_001.mem";  // MEM 파일 이름 추가
FILE* memOutFile = NULL;  // MEM 파일 포인터 추가
```

<br><br>
    
### 2. ======== MEM 파일 생성 부분 (추가) ========
__MEM 파일 생성__
```
if (fopen_s(&memOutFile, memFile, "w") != 0 || memOutFile == NULL) {
    printf("MEM 파일을 생성할 수 없습니다: %s\n", memFile);
    free(imageData);
    free(grayscaleData);
    return 1;
}
```

<br><br>
  
__MEM 파일에 그레이스케일 데이터를 16진수로 쓰기__
- 각 픽셀값을 2자리 16진수로 변환하고 줄바꿈(0x0D 0x0A)으로 구분
```
for (int i = 0; i < IMAGE_WIDTH * IMAGE_HEIGHT; i++) {
    fprintf(memOutFile, "%02X\r\n", grayscaleData[i]);
}

fclose(memOutFile);
```

<br><br>
  
### 3. ========== MEM 파일 생성 완료 메시지 ================
```
printf("Verilog MEM 파일: %s\n", memFile);  // MEM 파일 생성 완료 메시지 추가
```

<br>

# BMP 그레이스케일 이미지 MEM 변환

<br>

__예시__

![output_edge_map.bmp](https://github.com/user-attachments/files/22039548/output_edge_map.bmp)

<br>

- 소벨 필터
```
int sobelX[3][3] = {
    {-1, 0, 1},
    {-2, 0, 2},
    {-1, 0, 1}
};
int sobelY[3][3] = {
    {-1, -2, -1},
    { 0,  0,  0},
    { 1,  2,  1}
};
```

<br><br>

- 안전하게 픽셀 값 가져오기
```
unsigned char getPixel(unsigned char* image, int width, int height, int x, int y) {
    if (x < 0) x = 0;
    if (x >= width) x = width - 1;
    if (y < 0) y = 0;
    if (y >= height) y = height - 1;
    return image[y * width + x];
}
```

<br><br>

- 소벨 필터로 엣지 찾기
```
void findEdges(unsigned char* input, unsigned char* output, int width, int height) {
    printf("엣지 검출 중...\n");
    for (int y = 0; y < height; y++) {
        for (int x = 0; x < width; x++) {
            int gx = 0, gy = 0;
            // 주변 9개 픽셀에 필터 적용
            for (int i = -1; i <= 1; i++) {
                for (int j = -1; j <= 1; j++) {
                    unsigned char pixel = getPixel(input, width, height, x + j, y + i);
                    gx += pixel * sobelX[i + 1][j + 1];
                    gy += pixel * sobelY[i + 1][j + 1];
                }
            }
            // 엣지 강도 계산
            int edge = (int)sqrt(gx * gx + gy * gy);
            if (edge > 255) edge = 255;
            output[y * width + x] = edge;
        }
    }
}

const char* edgeFile = "output_edge.bmp";        // 엣지 검출 BMP 파일 추가
FILE* edgeOutFile = NULL;                        // 엣지 검출 BMP 파일 포인터 추가
```

<br><br>

// 3. 그레이스케일 변환 후 엣지 검출 추가
```
for (int i = 0; i < IMAGE_WIDTH * IMAGE_HEIGHT; i++) {
    grayscaleData[i] = rgbToGrayscale(imageData[i]);
}

// 엣지 검출을 위한 메모리 할당 및 처리
uint8_t* edgeData = (uint8_t*)malloc(IMAGE_WIDTH * IMAGE_HEIGHT);
if (edgeData == NULL) {
    printf("엣지 데이터 메모리 할당 실패\n");
    free(imageData);
    free(grayscaleData);
    return 1;
}

// 소벨 필터로 엣지 검출

findEdges(grayscaleData, edgeData, IMAGE_WIDTH, IMAGE_HEIGHT);
```

<br><br>

// 4. 엣지 검출 BMP 파일 저장
// 엣지 검출 결과를 BMP로 저장
```
if (fopen_s(&edgeOutFile, edgeFile, "wb") != 0 || edgeOutFile == NULL) {
    printf("엣지 출력 파일을 생성할 수 없습니다: %s\n", edgeFile);
    free(imageData);
    free(grayscaleData);
    free(edgeData);
    return 1;
}

// 엣지 검출 BMP 헤더는 그레이스케일과 동일

fwrite(&newFileHeader, sizeof(BMPFileHeader), 1, edgeOutFile);
fwrite(&newInfoHeader, sizeof(BMPInfoHeader), 1, edgeOutFile);

// 그레이스케일 팔레트 쓰기

for (int i = 0; i < 256; i++) {
    uint8_t color[4] = { i, i, i, 0 };
    fwrite(color, 4, 1, edgeOutFile);
}

// 엣지 데이터 쓰기

for (int y = IMAGE_HEIGHT - 1; y >= 0; y--) {
    for (int x = 0; x < IMAGE_WIDTH; x++) {
        fwrite(&edgeData[y * IMAGE_WIDTH + x], 1, 1, edgeOutFile);
    }
    if (newPadding > 0) {
        fwrite(paddingBytes, newPadding, 1, edgeOutFile);
    }
}
fclose(edgeOutFile);

// 엣지 데이터 메모리 해제 추가
free(edgeData);
```

<br><br>

7. 완료 메시지 수정
```
printf("엣지 검출 BMP 파일: %s\n", edgeFile);        // 엣지 BMP 파일 메시지 추가
```






