# 💌 AppCenterChallenge- By. Seohyun-Kim  

[**앱센터 과제 소개 깃허브**](https://github.com/inu-appcenter/challenge-13-5)  
<br>  

## JSON TEST FILE (입력)  
students.json  
```
[
  {
    "name": "샌액희",
    "studentId": "202199999"
  },
  {
    "name": "개발자",
    "studentId": "202099999"
  },
  {
    "name": "김서현",
    "major": "Embedded System Engineering",
    "studentId": "201901739",
    "goal": "Cool developer!"
  }
]
```

## 출력 결과  

![image](https://user-images.githubusercontent.com/61939286/132104866-b59427f8-e7f4-47af-b174-8e9781bc5f43.png)


## 주안점   

* 파일 입출력을 통해 json파일 읽어 버퍼에 저장
* 저장된 문자열에서 json형태의 시작인 '{'이 나왔을때 문자열 " " 안의 값을 TOKEN으로 저장
* KEY: VALUE 형태이므로 tokenIndex의 홀짝 여부에 따라 출력 달리함
* {} 로 묶여있는 한 객체(Object)가 끝날 때마다 줄바꿈

<br>  


## 코드 설명  
 ### 1. 구조체 선언  
 여기서는 String 형태만 사용합니다.
 ```c
 #define TOKEN_COUNT 100    // 토큰의 최대 개수
 
 // 토큰 종류 열거형
typedef enum _TOKEN_TYPE {
	TOKEN_STRING,    // 문자열 토큰
	TOKEN_NUMBER,    // 숫자 토큰
} TOKEN_TYPE;

// 토큰 구조체
typedef struct _TOKEN {
	TOKEN_TYPE type;   // 토큰 종류
	union {            // 두 종류 중 한 종류만 저장할 것이므로 공용체로 만듦
		char* string;     // 문자열 포인터
		double number;    // 실수형 숫자
	};
	bool isArray;      // 현재 토큰이 배열인지 표시
} TOKEN;

// JSON 구조체
typedef struct _JSON {
	TOKEN tokens[TOKEN_COUNT]; // 토큰 배열
} JSON;
```  

### 2. 파일 읽어 내용을 문자열로 반환 함수  
```c
char* readFile(char* filename, int* readSize)    // 파일을 읽어서 내용을 반환하는 함수
{
	FILE* fp = fopen(filename, "rb");
	if (fp == NULL)
		return NULL;

	int size;
	char* buffer;

	// 파일 크기 구하기
	fseek(fp, 0, SEEK_END);
	size = ftell(fp);
	fseek(fp, 0, SEEK_SET);

	// 파일 크기 + NULL 공간만큼 메모리를 할당하고 0으로 초기화
	buffer = malloc(size + 1);
	memset(buffer, 0, size + 1);

	// 파일 내용 읽기
	if (fread(buffer, size, 1, fp) < 1)
	{
		*readSize = 0;
		free(buffer);
		fclose(fp);
		return NULL;
	}

	// 파일 크기를 넘겨줌
	*readSize = size;

	fclose(fp);    // 파일 포인터 닫기

	return buffer;
}
```  

### 3. JSON 파싱 함수  

![image](https://user-images.githubusercontent.com/61939286/132105507-697646b7-7e43-46d2-b671-7f528cd63de7.png)

```c
void parseJSON(char* doc, int size, JSON* json)    // JSON 파싱 함수
{
	int tokenIndex = 0;    // 토큰 인덱스
	int pos = 0;           // 문자 검색 위치를 저장하는 변수

	while (doc[pos] != '{')// {이 나올 때까지 위치 이동
	{
		pos++;
	} 1️⃣

	pos++;    // 다음 문자로

	while (pos < size)       // 문서 크기만큼 반복
	{
		switch (doc[pos])    // 문자의 종류에 따라 분기
		{
			
			case '"':   2️⃣         // 문자가 "이면 문자열
			{
				// 문자열의 시작 위치를 구함. 맨 앞의 "를 제외하기 위해 + 1
				char* begin = doc + pos + 1;

				// 문자열의 끝 위치를 구함. 다음 "의 위치
				char* end = strchr(begin, '"');
				if (end == NULL)    // "가 없으면 잘못된 문법이므로 
					break;          // 반복을 종료

				int stringLength = end - begin;    // 문자열의 실제 길이는 끝 위치 - 시작 위치

				// 토큰 배열에 문자열 저장
				json->tokens[tokenIndex].type = TOKEN_STRING;
				// 문자열 길이 + NULL 공간만큼 메모리 할당
				json->tokens[tokenIndex].string = malloc(stringLength + 1);
				// 할당한 메모리를 0으로 초기화
				memset(json->tokens[tokenIndex].string, 0, stringLength + 1);

				// 문서에서 문자열을 토큰에 저장
				// 문자열 시작 위치에서 문자열 길이만큼만 복사
				memcpy(json->tokens[tokenIndex].string, begin, stringLength);

				if (tokenIndex % 2 == 0) // Key 이면
				{
					printf("%s: ", json->tokens[tokenIndex].string);
				}
				else // Value 이면
				{
					printf("%s\n", json->tokens[tokenIndex].string);
				}				
				tokenIndex++; // 토큰 인덱스 증가

				pos = pos + stringLength + 1;    // 현재 위치 + 문자열 길이 + "(+ 1)
				3️⃣
			}
			break;

		}

		if (doc[pos] == '}') { //한 그룹 종료4️⃣
			printf("\n");
		}

		pos++; // 다음 문자로
	}


}
```  

### 4. JSON구조체 형식으로 할당된 변수 해제해주는 함수  
```c
void freeJSON(JSON* json)    // JSON 해제 함수
{
	for (int i = 0; i < TOKEN_COUNT; i++)            // 토큰 개수만큼 반복
	{
		if (json->tokens[i].type == TOKEN_STRING)    // 토큰 종류가 문자열이면
			free(json->tokens[i].string);            // 동적 메모리 해제
	}
}
```  

### 5. Main 함수  
```c
int main()
{
	int size; // 문서 크기

	char* doc = readFile("students.json", &size);    // 파일에서 JSON 문서를 읽음, 문서 크기를 구함
	if (doc == NULL)
		return -1;

	JSON json = { 0, };             // JSON 구조체 변수 선언 및 초기화

	parseJSON(doc, size, &json);    // JSON 문서 파싱

	freeJSON(&json);    // json에 할당된 동적 메모리 해제

	free(doc);    // 문서 동적 메모리 해제

	return 0;
}
```
