<h1 align="center">ð C++ Primer 0x08 ç»ä¹ é¢è§£</h1>

## 8.1 IOç±»

### 8.1.2 æ¡ä»¶ç¶æ

> 8.1 ç¼åå½æ°ï¼æ¥åä¸ä¸ª`istream&`åæ°ï¼è¿åå¼ç±»åä¹æ¯`istream&`ãæ­¤å½æ°é¡»ä»ç»å®æµä¸­è¯»åæ°æ®ï¼ç´è³éå°æä»¶ç»ææ è¯æ¶åæ­¢ãå®å°è¯»åçæ°æ®æå°å¨æ åè¾åºä¸ãå®æè¿äºæä½åï¼å¨è¿åæµä¹åï¼å¯¹æµè¿è¡å¤ä½ï¼ä½¿å¶å¤äºææç¶æã
>
> 8.2 æµè¯å½æ°ï¼è°ç¨åæ°ä¸º`cin`ã

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



> 8.3 ä»ä¹æåµä¸ï¼ä¸é¢ç`while`å¾ªç¯ä¼ç»æ­¢ï¼

```cpp
while (cin >> i) /*  ...    */
```

* å¦æ`badbit`ã`failbit`å`eofbit`ä»»ä¸è¢«ç½®ä½ï¼æ£æµæµç¶æçæ¡ä»¶å°±ä¼å¤±è´¥



## 8.2 æä»¶è¾å¥è¾åº

> 8.4 ç¼åå½æ°ï¼ä»¥è¯»æ¨¡å¼æå¼ä¸ä¸ªæä»¶ï¼å°å¶åå®¹è¯»å¥å°ä¸ä¸ª`string`ç`vector`ä¸­ï¼å°æ¯ä¸è¡ä½ä¸ºä¸ä¸ªç¬ç«çåç´ å­äº`vector`ä¸­ã

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



> 8.5 éåä¸é¢çç¨åºï¼å°æ¯ä¸ªåè¯ä½ä¸ºä¸ä¸ªç¬ç«çåç´ è¿è¡å­å¨ã 

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



> 8.6 éå7.1.1èçä¹¦åºç¨åºï¼ä»ä¸ä¸ªæä»¶ä¸­è¯»åäº¤æè®°å½ãå°æä»¶åä½ä¸ºä¸ä¸ªåæ°ä¼ éç»`main`ã

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



### 8.2.2 æä»¶æ¨¡å¼

> 8.7 ä¿®æ¹ä¸ä¸èçä¹¦åºç¨åºï¼å°ç»æä¿å­å°ä¸ä¸ªæä»¶ä¸­ãå°è¾åºæä»¶åä½ä¸ºç¬¬äºä¸ªåæ°ä¼ éç»`main`å½æ°ã

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



> 8.8 ä¿®æ¹ä¸ä¸é¢çç¨åºï¼å°ç»æè¿½å å°ç»å®çæä»¶æ«å°¾ãå¯¹åä¸ä¸ªè¾åºæä»¶ï¼è¿è¡ç¨åºè³å°ä¸¤æ¬¡ï¼æ£éªæ°æ®æ¯å¦å¾ä»¥ä¿çã

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



## 8.8 string æµ

### 8.3.1 ä½¿ç¨ istringstream

> 8.9 ä½¿ç¨ä½ ä¸º8.1.2èç¬¬ä¸ä¸ªç»ä¹ æç¼åçå½æ°æå°ä¸ä¸ª`istringstream`å¯¹è±¡çåå®¹ã

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



> 8.10 ç¼åç¨åºï¼å°æ¥èªä¸ä¸ªæä»¶ä¸­çè¡ä¿å­å¨ä¸ä¸ª`vector<string>`ä¸­ãç¶åä½¿ç¨ä¸ä¸ª`istringstream`ä»`vector`è¯»åæ°æ®åç´ ï¼æ¯æ¬¡è¯»åä¸ä¸ªåè¯ã

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



> 8.11 æ¬èçç¨åºå¨å¤å±`while`å¾ªç¯ä¸­å®ä¹äº`istringstream`å¯¹è±¡ãå¦æ`record`å¯¹è±¡å®ä¹å¨å¾ªç¯ä¹å¤ï¼ä½ éè¦å¯¹ç¨åºè¿è¡ææ ·çä¿®æ¹ï¼éåç¨åºï¼å°`record`çå®ä¹ç§»å°`while`å¾ªç¯ä¹å¤ï¼éªè¯ä½ è®¾æ³çä¿®æ¹æ¹æ³æ¯å¦æ­£ç¡®ã

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



> 8.12 æä»¬ä¸ºä»ä¹æ²¡æå¨`PersonInfo`ä¸­ä½¿ç¨ç±»ååå§åï¼

åªéè¦èåç±»å³å¯ï¼ä¸éè¦ç±»ååå§å



### 8.3.2 ä½¿ç¨ ostringstream

> 8.13 éåæ¬èççµè¯å·ç ç¨åºï¼ä»ä¸ä¸ªå½åæä»¶èé`cin`è¯»åæ°æ®ã

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



> 8.14 æä»¬ä¸ºä»ä¹å°`entry`å`nums`å®ä¹ä¸º`const auto&`ï¼

ä¸éè¦æ¹åï¼æä»¥ç¨`const`

å®ä»¬æ¯ç±»ç±»åï¼ä½¿ç¨å¼ç¨å¯ä»¥é¿åæ·è´