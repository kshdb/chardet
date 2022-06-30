# chardet
文本编码自动推断，将常见的编码汇总

# 应用示例
```go

package main

import (
	"fmt"
	"io"
	"os"
	"path/filepath"

	"github.com/kshdb/chardet"
)

var (
	zh_gb18030_text = []byte{
		71, 111, 202, 199, 71, 111, 111, 103, 108, 101, 233, 95, 176, 108, 181, 196, 210, 187, 214, 214, 190, 142, 215, 103, 208, 205, 163, 172, 129, 75, 176, 108,
		208, 205, 163, 172, 178, 162, 190, 223, 211, 208, 192, 172, 187, 248, 187, 216, 202, 213, 185, 166, 196, 220, 181, 196, 177, 224, 179, 204, 211, 239, 209, 212,
		161, 163, 10,
	}
	zh_utf8_text    = []byte("%u4E2D%u7405%u8F6F%u4EF6")
	zh_unicode_text = []byte("\u4e2d\u7405\u8f6f\u4ef6")
)

func main() {
	detector := chardet.NewTextDetector()
	result, err := detector.DetectBest(zh_gb18030_text)
	if err == nil {
		fmt.Printf(
			"编码是 %s, 语言是 %s\n",
			result.Charset,
			result.Language)
	} else {
		fmt.Println("出错了", err.Error())
	}

	result, err = detector.DetectBest(zh_utf8_text)
	if err == nil {
		fmt.Printf(
			"编码是 %s, 语言是 %s\n",
			result.Charset,
			result.Language)
	} else {
		fmt.Println("出错了", err.Error())
	}

	result, err = detector.DetectBest(zh_unicode_text)
	if err == nil {
		fmt.Printf(
			"编码是 %s, 语言是 %s\n",
			result.Charset,
			result.Language)
	} else {
		fmt.Println("出错了", err.Error())
	}

	test_file1()
}

func test_file() {
	type file_charset_language struct {
		File     string
		IsHtml   bool
		Charset  string
		Language string
	}
	var data = []file_charset_language{
		{"utf8.html", true, "UTF-8", ""},
		{"utf8_bom.html", true, "UTF-8", ""},
		{"8859_1_en.html", true, "ISO-8859-1", "en"},
		{"8859_1_da.html", true, "ISO-8859-1", "da"},
		{"8859_1_de.html", true, "ISO-8859-1", "de"},
		{"8859_1_es.html", true, "ISO-8859-1", "es"},
		{"8859_1_fr.html", true, "ISO-8859-1", "fr"},
		{"8859_1_pt.html", true, "ISO-8859-1", "pt"},
		{"shift_jis.html", true, "Shift_JIS", "ja"},
		{"gb18030.html", true, "GB-18030", "zh"},
		{"euc_jp.html", true, "EUC-JP", "ja"},
		{"euc_kr.html", true, "EUC-KR", "ko"},
		{"big5.html", true, "Big5", "zh"},
	}

	textDetector := chardet.NewTextDetector()
	htmlDetector := chardet.NewHtmlDetector()
	buffer := make([]byte, 32<<10)
	for _, d := range data {
		f, err := os.Open(filepath.Join("./testdata", d.File))
		if err != nil {
			fmt.Println("出错了", err.Error())
		}
		defer f.Close()
		size, _ := io.ReadFull(f, buffer)
		input := buffer[:size]
		var detector = textDetector
		if d.IsHtml {
			detector = htmlDetector
		}
		result, err := detector.DetectBest(input)
		if err != nil {
			fmt.Println("出错了", err.Error())
		}
		//if result.Charset != d.Charset {
		fmt.Printf("%s--既定编码格式是：%s, 推断格式是： %s\n", d.File, d.Charset, result.Charset)
		// }
		// if result.Language != d.Language {
		// 	fmt.Printf("Expected language %s, actual %s", d.Language, result.Language)
		// }
	}
}

func test_file1() {
	type file_charset_language struct {
		File     string
		IsHtml   bool
		Charset  string
		Language string
	}
	var data = []file_charset_language{
		{"unicode big.txt", false, "UTF-16BE", ""},
		{"unicode.txt", false, "UTF-16LE", ""},
		{"UTF8.txt", false, "UTF-8", "en"},
		{"UTF8BOM.txt", false, "UTF-8BOM", ""},
		{"UTF16BE.txt", false, "UTF-16BE", ""},
		{"UTF16LE.txt", false, "UTF-16LE", ""},
		{"数据.txt", false, "UTF-8", ""},
		{"ANSI.txt", false, "ANSI", ""},
	}

	textDetector := chardet.NewTextDetector()
	htmlDetector := chardet.NewHtmlDetector()
	buffer := make([]byte, 32<<10)
	for _, d := range data {
		f, err := os.Open(filepath.Join("./testtxt", d.File))
		if err != nil {
			fmt.Println("出错了", err.Error())
		}
		defer f.Close()
		size, _ := io.ReadFull(f, buffer)
		input := buffer[:size]
		var detector = textDetector
		if d.IsHtml {
			detector = htmlDetector
		}
		result, err := detector.DetectBest(input)
		if err != nil {
			fmt.Println("出错了", err.Error())
		}
		//if result.Charset != d.Charset {
		fmt.Printf("%s--既定编码格式是：%s, 推断格式是： %s\n", d.File, d.Charset, result.Charset)
		// }
		// if result.Language != d.Language {
		// 	fmt.Printf("Expected language %s, actual %s", d.Language, result.Language)
		// }
	}
}


```