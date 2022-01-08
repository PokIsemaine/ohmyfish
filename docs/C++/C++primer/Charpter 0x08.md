<h1 align="center">📔 C++ Primer 0x08 练习题解</h1>

## 8.1 IO类

### 8.1.2 条件状态

> 8.1 编写函数，接受一个`istream&`参数，返回值类型也是`istream&`。此函数须从给定流中读取数据，直至遇到文件结束标识时停止。它将读取的数据打印在标准输出上。完成这些操作后，在返回流之前，对流进行复位，使其处于有效状态。
>
> 8.2 测试函数，调用参数为`cin`。

```cpp
#include <iostream>

std::istream& fun(std::istream& is){
	std::string buf;
	while(is >> buf){
		std::cout << buf << std::endl;	
	}
	is.clear();
	return is;
}

int main(){
	std::istream& is = fun(std::cin);
	std::cout << is.rdstate() << std::endl;
	return 0;
}
```



> 8.3 什么情况下，下面的`while`循环会终止？

```cpp
while (cin >> i) /*  ...    */
```

* 如果`badbit`、`failbit`和`eofbit`任一被置位，检测流状态的条件就会失败



## 8.2 文件输入输出

> 8.4 编写函数，以读模式打开一个文件，将其内容读入到一个`string`的`vector`中，将每一行作为一个独立的元素存于`vector`中。

```cpp
void read(const std::string& ifile,std::vector<std::string>& vec){
	std::ifstream input(ifile);
	if(input){
		std::string buf;
		while(geline(input,buf)){
			vec.push_back(buf);
		}
	}	
}
```



> 8.5 重写上面的程序，将每个单词作为一个独立的元素进行存储。 

```cpp
void read(const std::string& ifile,std::vector<std::string>& vec){
	std::ifstream input(ifile);
	if(input){
		std::string buf;
		while(input >> buf){
			vec.push_back(buf);
		}
	}	
}
```



> 8.6 重写7.1.1节的书店程序，从一个文件中读取交易记录。将文件名作为一个参数传递给`main`。

```cpp
#include <iostream>
#include <string>
#include <fstream>

struct Sales_data {
	std::string bookNo;
	unsigned units_sold = 0;
	double revenue = 0.0;
};

int main(int argc,char **argv){
	std::ifstream input(argv[1]);
	
	Sales_data total,item;
    double price = 0;
    if(input){
    	if (input >> total.bookNo >> total.units_sold >> price){
        total.revenue = total.units_sold * price;
        while (input >> item.bookNo >> item.units_sold >> price){
            item.revenue = item.units_sold * price;
            if (total.bookNo == item.bookNo) {
                total.units_sold += item.units_sold;
                total.revenue += item.revenue;
            }else {
                std::cout << total.bookNo << " " 
                		<< total.units_sold << " "
                		<< total.revenue << std::endl;
                total = item;
            }
        }
        std::cout << total.bookNo << " " 
				<< total.units_sold << " "
				<< total.revenue << std::endl;
	    }else {
	        std::cerr << "No data?!" << std::endl;
	        return -1;
	    }
    }
    
	return 0;
}
```



### 8.2.2 文件模式

> 8.7 修改上一节的书店程序，将结果保存到一个文件中。将输出文件名作为第二个参数传递给`main`函数。

```cpp
#include <iostream>
#include <string>
#include <fstream>

struct Sales_data {
	std::string bookNo;
	unsigned units_sold = 0;
	double revenue = 0.0;
};

int main(int argc,char **argv){
	std::ifstream input(argv[1]);
	std::ofstream output(argv[2]);
	Sales_data total,item;
    double price = 0;
    if(input){
    	if (input >> total.bookNo >> total.units_sold >> price){
        total.revenue = total.units_sold * price;
        while (input >> item.bookNo >> item.units_sold >> price){
            item.revenue = item.units_sold * price;
            if (total.bookNo == item.bookNo) {
                total.units_sold += item.units_sold;
                total.revenue += item.revenue;
            }else {
                output << total.bookNo << " " 
                		<< total.units_sold << " "
                		<< total.revenue << std::endl;
                total = item;
            }
        }
        output << total.bookNo << " " 
				<< total.units_sold << " "
				<< total.revenue << std::endl;
	    }else {
	        std::cerr << "No data?!" << std::endl;
	        return -1;
	    }
    }
    
	return 0;
}
```



> 8.8 修改上一题的程序，将结果追加到给定的文件末尾。对同一个输出文件，运行程序至少两次，检验数据是否得以保留。

```cpp
#include <iostream>
#include <istream>
#include <string>
#include <fstream>

struct Sales_data {
        std::string bookNo;
        unsigned units_sold = 0;
        double revenue = 0.0;
};

int main(int argc,char **argv){
    std::ifstream input(argv[1]);
    std::ofstream output(argv[2],std::ofstream::app);
    Sales_data total,item;
    double price = 0;
    if(input){
        if (input >> total.bookNo >> total.units_sold >> price){
        total.revenue = total.units_sold * price;
        while (input >> item.bookNo >> item.units_sold >> price){
            item.revenue = item.units_sold * price;
            if (total.bookNo == item.bookNo) {
                total.units_sold += item.units_sold;
                total.revenue += item.revenue;
            }else {
                output  << total.bookNo << " " 
                                << total.units_sold << " "
                                << total.revenue << std::endl;
                total = item;
            }
        }
        output << total.bookNo << " " 
                                << total.units_sold << " "
                                << total.revenue << std::endl;
            }else {
                std::cerr << "No data?!" << std::endl;
                return -1;
            }
    }
    
    return 0;
}
```



## 8.8 string 流

### 8.3.1 使用 istringstream

> 8.9 使用你为8.1.2节第一个练习所编写的函数打印一个`istringstream`对象的内容。

```cpp
#include <iostream>
#include <sstream>

std::istream& fun(std::istream& is){
	std::string buf;
	while(is >> buf){
		std::cout << buf << std::endl;	
	}
	is.clear();
	return is;
}

int main(){
	std::string line;
	getline(std::cin,line);
	std::istringstream record(line);
	std::istream& is = fun(record);
	std::cout << is.rdstate() << std::endl;
	return 0;
}
```



> 8.10 编写程序，将来自一个文件中的行保存在一个`vector<string>`中。然后使用一个`istringstream`从`vector`读取数据元素，每次读取一个单词。

```cpp
#include <iostream>
#include <sstream>
#include <vector>

int main(){
	std::string line,word;
	std::vector<std::string>vec;
	while(getline(std::cin,line)){
		std::istringstream record(line);
		if(record){
			while(record >> word){
				vec.push_back(word);
			}
		}
		
	}
	for(auto v:vec)std::cout << v << std::endl;
	
	return 0;
}
```



> 8.11 本节的程序在外层`while`循环中定义了`istringstream`对象。如果`record`对象定义在循环之外，你需要对程序进行怎样的修改？重写程序，将`record`的定义移到`while`循环之外，验证你设想的修改方法是否正确。

```cpp
#include <iostream>
#include <sstream>
#include <string>
#include <vector>

struct PersonInfo {
    std::string name;
    std::vector<std::string> phones;
};

int main(){
    std::string line, word;
    std::vector<PersonInfo> people;
    std::istringstream record;
    while (getline(std::cin, line)){
        PersonInfo info;
        record.clear();record.str(line);
        record >> info.name;
        while (record >> word)
            info.phones.push_back(word);
        people.push_back(info);
    }
    
    for (auto &p : people){
        std::cout << p.name << " ";
        for (auto &s : p.phones)
            std::cout << s << " ";
        std::cout << std::endl;
    }
    
    return 0;
}
```



> 8.12 我们为什么没有在`PersonInfo`中使用类内初始化？

只需要聚合类即可，不需要类内初始化



### 8.3.2 使用 ostringstream

> 8.13 重写本节的电话号码程序，从一个命名文件而非`cin`读取数据。

```cpp
#include <iostream>
#include <sstream>
#include <string>
#include <vector>
#include <fstream>

struct PersonInfo {
    std::string name;
    std::vector<std::string> phones;
};

bool vaild(const std::string& str){
	return isdigit(str[0]);
}

std::string format(const std::string& str){
	return str.substr(0,3) + "-" + str.substr(3,3) + "-" + str.substr(6);
}
int main(int argc,char **argv){
	
	std::ifstream input(argv[1]);
    std::string line, word;
    std::vector<PersonInfo> people;
    std::istringstream record;
    
    while (getline(input, line)){
        PersonInfo info;
        record.clear();record.str(line);
        record >> info.name;
        while (record >> word)
            info.phones.push_back(word);
        people.push_back(info);
    }
    
    for(const auto &entry:people){
    	std::ostringstream formatted,badNums;
    	for(const auto &nums:entry.phones){
    		if(!vaild(nums)){
    			badNums << " " << nums;
    		}else{
    			formatted << " " << nums;
    		}
    	}
    	if(badNums.str().empty())
    		std::cout << entry.name << " " << formatted.str() << std::endl;
    	else
    		std::cerr << "input error: " << entry.name
    			<< "invaild number(s) " << badNums.str() << std::endl;
    }
    
    return 0;
}
```



> 8.14 我们为什么将`entry`和`nums`定义为`const auto&`？

不需要改变，所以用`const`

它们是类类型，使用引用可以避免拷贝