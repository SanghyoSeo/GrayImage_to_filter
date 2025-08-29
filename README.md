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

예시

![output_edge_001.bmp](https://github.com/user-attachments/files/22039404/output_edge_001.bmp)

<br>

// 소벨 필터
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

// 안전하게 픽셀 값 가져오기
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

// 소벨 필터로 엣지 찾기
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
```
<br><br>

```
const char* edgeFile = "output_edge.bmp";        // 엣지 검출 BMP 파일 추가
FILE* edgeOutFile = NULL;                        // 엣지 검출 BMP 파일 포인터 추가
```

<br><br>

// 3. 그레이스케일 변환 후 엣지 검출 추가
```
for (int i = 0; i < IMAGE_WIDTH * IMAGE_HEIGHT; i++) {
    grayscaleData[i] = rgbToGrayscale(imageData[i]);
}
```

<br><br>

// 엣지 검출을 위한 메모리 할당 및 처리
```
uint8_t* edgeData = (uint8_t*)malloc(IMAGE_WIDTH * IMAGE_HEIGHT);
if (edgeData == NULL) {
    printf("엣지 데이터 메모리 할당 실패\n");
    free(imageData);
    free(grayscaleData);
    return 1;
}
```

<br><br>

// 소벨 필터로 엣지 검출
```
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
```

<br><br>

// 엣지 검출 BMP 헤더는 그레이스케일과 동일
```
fwrite(&newFileHeader, sizeof(BMPFileHeader), 1, edgeOutFile);
fwrite(&newInfoHeader, sizeof(BMPInfoHeader), 1, edgeOutFile);
```

<br><br>

// 그레이스케일 팔레트 쓰기
```
for (int i = 0; i < 256; i++) {
    uint8_t color[4] = { i, i, i, 0 };
    fwrite(color, 4, 1, edgeOutFile);
}
```

<br><br>

// 엣지 데이터 쓰기
```
for (int y = IMAGE_HEIGHT - 1; y >= 0; y--) {
    for (int x = 0; x < IMAGE_WIDTH; x++) {
        fwrite(&edgeData[y * IMAGE_WIDTH + x], 1, 1, edgeOutFile);
    }
    if (newPadding > 0) {
        fwrite(paddingBytes, newPadding, 1, edgeOutFile);
    }
}
fclose(edgeOutFile);
```

<br><br>

// 엣지 데이터 메모리 해제 추가
```
free(edgeData);
```

<br><br>

// 7. 완료 메시지 수정
```
printf("엣지 검출 BMP 파일: %s\n", edgeFile);        // 엣지 BMP 파일 메시지 추가
```






