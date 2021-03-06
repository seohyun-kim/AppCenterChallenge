# ๐ AppCenterChallenge- By. Seohyun-Kim  

[**์ฑ์ผํฐ ๊ณผ์  ์๊ฐ ๊นํ๋ธ**](https://github.com/inu-appcenter/challenge-13-5)  
<br>  

## JSON TEST FILE (์๋ ฅ)  
students.json  
```
[
  {
    "name": "์์กํฌ",
    "studentId": "202199999"
  },
  {
    "name": "๊ฐ๋ฐ์",
    "studentId": "202099999"
  },
  {
    "name": "๊น์ํ",
    "major": "Embedded System Engineering",
    "studentId": "201901739",
    "goal": "Cool developer!"
  }
]
```

## ์ถ๋ ฅ ๊ฒฐ๊ณผ  

![image](https://user-images.githubusercontent.com/61939286/132104866-b59427f8-e7f4-47af-b174-8e9781bc5f43.png)


## ์ฃผ์์    

* ํ์ผ ์์ถ๋ ฅ์ ํตํด jsonํ์ผ ์ฝ์ด ๋ฒํผ์ ์ ์ฅ
* ์ ์ฅ๋ ๋ฌธ์์ด์์ jsonํํ์ ์์์ธ '{'์ด ๋์์๋ ๋ฌธ์์ด " " ์์ ๊ฐ์ TOKEN์ผ๋ก ์ ์ฅ
* KEY: VALUE ํํ์ด๋ฏ๋ก tokenIndex์ ํ์ง ์ฌ๋ถ์ ๋ฐ๋ผ ์ถ๋ ฅ ๋ฌ๋ฆฌํจ
* {} ๋ก ๋ฌถ์ฌ์๋ ํ ๊ฐ์ฒด(Object)๊ฐ ๋๋  ๋๋ง๋ค ์ค๋ฐ๊ฟ

<br>  


## ์ฝ๋ ์ค๋ช  
[์ฐธ๊ณ ํ ๋ฌธ์](https://dojang.io/mod/page/view.php?id=724)  

 ### 1. ๊ตฌ์กฐ์ฒด ์ ์ธ  
 ์ฌ๊ธฐ์๋ String ํํ๋ง ์ฌ์ฉํฉ๋๋ค.
 ```c
 #define TOKEN_COUNT 100    // ํ ํฐ์ ์ต๋ ๊ฐ์
 
 // ํ ํฐ ์ข๋ฅ ์ด๊ฑฐํ
typedef enum _TOKEN_TYPE {
	TOKEN_STRING,    // ๋ฌธ์์ด ํ ํฐ
	TOKEN_NUMBER,    // ์ซ์ ํ ํฐ
} TOKEN_TYPE;

// ํ ํฐ ๊ตฌ์กฐ์ฒด
typedef struct _TOKEN {
	TOKEN_TYPE type;   // ํ ํฐ ์ข๋ฅ
	union {            // ๋ ์ข๋ฅ ์ค ํ ์ข๋ฅ๋ง ์ ์ฅํ  ๊ฒ์ด๋ฏ๋ก ๊ณต์ฉ์ฒด๋ก ๋ง๋ฆ
		char* string;     // ๋ฌธ์์ด ํฌ์ธํฐ
		double number;    // ์ค์ํ ์ซ์
	};
	bool isArray;      // ํ์ฌ ํ ํฐ์ด ๋ฐฐ์ด์ธ์ง ํ์
} TOKEN;

// JSON ๊ตฌ์กฐ์ฒด
typedef struct _JSON {
	TOKEN tokens[TOKEN_COUNT]; // ํ ํฐ ๋ฐฐ์ด
} JSON;
```  

### 2. ํ์ผ ์ฝ์ด ๋ด์ฉ์ ๋ฌธ์์ด๋ก ๋ฐํ ํจ์  
```c
char* readFile(char* filename, int* readSize)    // ํ์ผ์ ์ฝ์ด์ ๋ด์ฉ์ ๋ฐํํ๋ ํจ์
{
	FILE* fp = fopen(filename, "rb");
	if (fp == NULL)
		return NULL;

	int size;
	char* buffer;

	// ํ์ผ ํฌ๊ธฐ ๊ตฌํ๊ธฐ
	fseek(fp, 0, SEEK_END);
	size = ftell(fp);
	fseek(fp, 0, SEEK_SET);

	// ํ์ผ ํฌ๊ธฐ + NULL ๊ณต๊ฐ๋งํผ ๋ฉ๋ชจ๋ฆฌ๋ฅผ ํ ๋นํ๊ณ  0์ผ๋ก ์ด๊ธฐํ
	buffer = malloc(size + 1);
	memset(buffer, 0, size + 1);

	// ํ์ผ ๋ด์ฉ ์ฝ๊ธฐ
	if (fread(buffer, size, 1, fp) < 1)
	{
		*readSize = 0;
		free(buffer);
		fclose(fp);
		return NULL;
	}

	// ํ์ผ ํฌ๊ธฐ๋ฅผ ๋๊ฒจ์ค
	*readSize = size;

	fclose(fp);    // ํ์ผ ํฌ์ธํฐ ๋ซ๊ธฐ

	return buffer;
}
```  

### 3. JSON ํ์ฑ ํจ์  

![image](https://user-images.githubusercontent.com/61939286/132105507-697646b7-7e43-46d2-b671-7f528cd63de7.png)

```c
void parseJSON(char* doc, int size, JSON* json)    // JSON ํ์ฑ ํจ์
{
	int tokenIndex = 0;    // ํ ํฐ ์ธ๋ฑ์ค
	int pos = 0;           // ๋ฌธ์ ๊ฒ์ ์์น๋ฅผ ์ ์ฅํ๋ ๋ณ์

	while (doc[pos] != '{')// {์ด ๋์ฌ ๋๊น์ง ์์น ์ด๋
	{
		pos++;
	} 1๏ธโฃ

	pos++;    // ๋ค์ ๋ฌธ์๋ก

	while (pos < size)       // ๋ฌธ์ ํฌ๊ธฐ๋งํผ ๋ฐ๋ณต
	{
		switch (doc[pos])    // ๋ฌธ์์ ์ข๋ฅ์ ๋ฐ๋ผ ๋ถ๊ธฐ
		{
			
			case '"':   2๏ธโฃ         // ๋ฌธ์๊ฐ "์ด๋ฉด ๋ฌธ์์ด
			{
				// ๋ฌธ์์ด์ ์์ ์์น๋ฅผ ๊ตฌํจ. ๋งจ ์์ "๋ฅผ ์ ์ธํ๊ธฐ ์ํด + 1
				char* begin = doc + pos + 1;

				// ๋ฌธ์์ด์ ๋ ์์น๋ฅผ ๊ตฌํจ. ๋ค์ "์ ์์น
				char* end = strchr(begin, '"');
				if (end == NULL)    // "๊ฐ ์์ผ๋ฉด ์๋ชป๋ ๋ฌธ๋ฒ์ด๋ฏ๋ก 
					break;          // ๋ฐ๋ณต์ ์ข๋ฃ

				int stringLength = end - begin;    // ๋ฌธ์์ด์ ์ค์  ๊ธธ์ด๋ ๋ ์์น - ์์ ์์น

				// ํ ํฐ ๋ฐฐ์ด์ ๋ฌธ์์ด ์ ์ฅ
				json->tokens[tokenIndex].type = TOKEN_STRING;
				// ๋ฌธ์์ด ๊ธธ์ด + NULL ๊ณต๊ฐ๋งํผ ๋ฉ๋ชจ๋ฆฌ ํ ๋น
				json->tokens[tokenIndex].string = malloc(stringLength + 1);
				// ํ ๋นํ ๋ฉ๋ชจ๋ฆฌ๋ฅผ 0์ผ๋ก ์ด๊ธฐํ
				memset(json->tokens[tokenIndex].string, 0, stringLength + 1);

				// ๋ฌธ์์์ ๋ฌธ์์ด์ ํ ํฐ์ ์ ์ฅ
				// ๋ฌธ์์ด ์์ ์์น์์ ๋ฌธ์์ด ๊ธธ์ด๋งํผ๋ง ๋ณต์ฌ
				memcpy(json->tokens[tokenIndex].string, begin, stringLength);

				if (tokenIndex % 2 == 0) // Key ์ด๋ฉด
				{
					printf("%s: ", json->tokens[tokenIndex].string);
				}
				else // Value ์ด๋ฉด
				{
					printf("%s\n", json->tokens[tokenIndex].string);
				}				
				tokenIndex++; // ํ ํฐ ์ธ๋ฑ์ค ์ฆ๊ฐ

				pos = pos + stringLength + 1;    // ํ์ฌ ์์น + ๋ฌธ์์ด ๊ธธ์ด + "(+ 1)
				3๏ธโฃ
			}
			break;

		}

		if (doc[pos] == '}') { //ํ ๊ทธ๋ฃน ์ข๋ฃ4๏ธโฃ
			printf("\n");
		}

		pos++; // ๋ค์ ๋ฌธ์๋ก
	}


}
```  

### 4. JSON๊ตฌ์กฐ์ฒด ํ์์ผ๋ก ํ ๋น๋ ๋ณ์ ํด์ ํด์ฃผ๋ ํจ์  
```c
void freeJSON(JSON* json)    // JSON ํด์  ํจ์
{
	for (int i = 0; i < TOKEN_COUNT; i++)            // ํ ํฐ ๊ฐ์๋งํผ ๋ฐ๋ณต
	{
		if (json->tokens[i].type == TOKEN_STRING)    // ํ ํฐ ์ข๋ฅ๊ฐ ๋ฌธ์์ด์ด๋ฉด
			free(json->tokens[i].string);            // ๋์  ๋ฉ๋ชจ๋ฆฌ ํด์ 
	}
}
```  

### 5. Main ํจ์  
```c
int main()
{
	int size; // ๋ฌธ์ ํฌ๊ธฐ

	char* doc = readFile("students.json", &size);    // ํ์ผ์์ JSON ๋ฌธ์๋ฅผ ์ฝ์, ๋ฌธ์ ํฌ๊ธฐ๋ฅผ ๊ตฌํจ
	if (doc == NULL)
		return -1;

	JSON json = { 0, };             // JSON ๊ตฌ์กฐ์ฒด ๋ณ์ ์ ์ธ ๋ฐ ์ด๊ธฐํ

	parseJSON(doc, size, &json);    // JSON ๋ฌธ์ ํ์ฑ

	freeJSON(&json);    // json์ ํ ๋น๋ ๋์  ๋ฉ๋ชจ๋ฆฌ ํด์ 

	free(doc);    // ๋ฌธ์ ๋์  ๋ฉ๋ชจ๋ฆฌ ํด์ 

	return 0;
}
```

[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2Fseohyun-kim%2FAppCenterChallenge&count_bg=%23EFBAF7&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)
