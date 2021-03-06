<h1 align="center">ð C++ Primer 0x0A ç»ä¹ é¢è§£</h1>

## 10.1 æ¦è¿°

> 10.1 å¤´æä»¶`algorithm`ä¸­å®ä¹äºä¸ä¸ªåä¸º`count`çå½æ°ï¼å®ç±»ä¼¼`find`ï¼ æ¥åä¸å¯¹è¿­ä»£å¨åä¸ä¸ªå¼ä½ä¸ºåæ°ã`count`è¿åç»å®å¼å¨åºåä¸­åºç°çæ¬¡æ°ãç¼åç¨åºï¼è¯»å`int`åºåå­å¥`vector`ä¸­ï¼æå°æå¤å°ä¸ªåç´ çå¼ç­äºç»å®å¼ã

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main(){
	std::vector<int>vec;

	int x;while(std::cin>>x)vec.push_back(x);

	std::cout << std::count(vec.cbegin(),vec.cend(),3);

	return 0;
}
```



> 10.2 éåä¸ä¸é¢ï¼ä½è¯»å `string` åºåå­å¥ `list` ä¸­ã

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>

int main(){
	std::list<std::string>ls;

	std::string x;
	while(std::cin>>x)ls.push_back(x);

	std::cout << std::count(ls.cbegin(),ls.cend(),std::string("abc"));

	return 0;
}
```



## 10.2 åè¯æ³åç®æ³

### 10.2.1 åªè¯»ç®æ³

> 10.3 ç¨ `accumulate`æ±ä¸ä¸ª `vector<int>` ä¸­åç´ ä¹åã

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate

int main(){
	std::vector<int>vec={1,2,3,4,5};
	int sum = std::accumulate(vec.cbegin(),vec.cend(),0);	
	std:: cout << sum;
	return 0;
}
```



> 10.4 åå® `v` æ¯ä¸ä¸ª`vector<double>`ï¼é£ä¹è°ç¨ `accumulate(v.cbegin(),v.cend(),0)` æä½éè¯¯ï¼å¦æå­å¨çè¯ï¼ï¼

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate

int main(){
	std::vector<double>vec={1.5,2.0,3.0,4.0,5.0};
	//ç´æ¥è¾åºçè¯ç»æintç±»å
	std:: cout << std::accumulate(vec.cbegin(),vec.cend(),0)<<std::endl;
	//ç¬¬ä¸ä¸ªåæ°å³å®ç±»å	
	std:: cout << std::accumulate(vec.cbegin(),vec.cend(),0.0);	
	
	return 0;
}
```



> 10.5 å¨æ¬èå¯¹ååï¼`roster`ï¼è°ç¨`equal`çä¾å­ä¸­ï¼å¦æä¸¤ä¸ªååä¸­ä¿å­çé½æ¯Cé£æ ¼å­ç¬¦ä¸²èä¸æ¯`string`ï¼ä¼åçä»ä¹ï¼

Cé£æ ¼å­ç¬¦ä¸²ä½¿ç¨æåå­ç¬¦æ°ç»çæéè¡¨ç¤ºï¼æä»¥æ¯è¾ä¸¤ä¸ªæéå­çå°åå¼ï¼èä¸æ¯åå®¹



### 10.2.2 åå®¹å¨åç´ çç®æ³

> 10.6 ç¼åç¨åºï¼ä½¿ç¨ `fill_n` å°ä¸ä¸ªåºåä¸­ç `int` å¼é½è®¾ç½®ä¸º0ã

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate

int main(){
	std::vector<int>vec={1,2,3,4,5};
	std::fill_n(vec.begin(),vec.size(),0);
	for(auto v:vec)std::cout << v << " ";
	
	return 0;
}
```



> 10.7 ä¸é¢ç¨åºæ¯å¦æéè¯¯ï¼å¦ææï¼è¯·æ¹æ­£ï¼

```cpp
(a) vector<int> vec; list<int> lst; int i;
	vec.resize(list.size());//å è¿å¥
	while (cin >> i)
		lst.push_back(i);
	copy(lst.cbegin(), lst.cend(), vec.begin());
(b) vector<int> vec;
	vec.reserve(10);//æ¹ævec.resize(10)
	fill_n(vec.begin(), 10, 0);
```

* æ·è´ç®æ³å°è¾å¥èå´ä¸­çåç´ æ·è´å°ç®çåºåä¸­ï¼ä¼ éç»`copy`çç®çåºåè³å°åå«åè¾å¥åºåä¸æ ·å¤çåç´ 
* ç®æ³ä¸æ£æ¥åæä½ï¼åç®çä½ç½®è¿­ä»£å¨åå¥æ°æ®çç®æ³åå®ç®çä½ç½®è¶³å¤å¤§ï¼è½å®¹çº³è¦åå¥çåç´ 



> 10.8 æ¬èæå°è¿ï¼æ ååºç®æ³ä¸ä¼æ¹åå®ä»¬ææä½çå®¹å¨çå¤§å°ãä¸ºä»ä¹ä½¿ç¨ `back_inserter`ä¸ä¼ä½¿è¿ä¸æ­è¨å¤±æï¼

`back_inserter` æ¯æå¥è¿­ä»£å¨ï¼å¨ `iterator.h` å¤´æä»¶ä¸­ï¼ä¸æ¯æ ååºçç®æ³ ï¼ï¼ï¼

æè§å¾`back_inserter`æ¬èº«ç¡®å®æ²¡ææ¹åææä½å®¹å¨çå¤§å°ï¼å®æ¯å°èµå¼è¿ç®ç¬¦åæè°ç¨`push_back()`ï¼æ¹åå®¹å¨çæ¯`push_back()`èä¸æ¯`back_inserter`,è¿æ ·è§£éæ¯è¾åç



### 10.2.3 éæå®¹å¨åç´ çç®æ³

> 10.9  å®ç°ä½ èªå·±ç`elimDups`ãåå«å¨è¯»åè¾å¥åãè°ç¨`unique`åä»¥åè°ç¨`erase`åæå°`vector`çåå®¹ã

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate

void print(const std::vector<std::string> &v){
	for(auto i:v)std::cout << i << " ";
	std::cout << std::endl;
}

void elimDups(std::vector<std::string> &words){
	std::sort(words.begin(),words.end());
	auto iter = std::unique(words.begin(),words.end());
	print(words);
	words.erase(iter,words.end());
}


int main(){
	std::vector<std::string>vs;
	std::string s;
	while(std::cin>>s)vs.push_back(s);
	print(vs);
	elimDups(vs);
	print(vs);
	return 0;
}
```



> 10.10 ä½ è®¤ä¸ºç®æ³ä¸æ¹åå®¹å¨å¤§å°çåå æ¯ä»ä¹ï¼

* è¿­ä»£å¨ä»¤ç®æ³ä¸ä¾èµäºå®¹å¨ï¼ä½ç®æ³ä¾èµäºåç´ ç±»åçæä½
* æ³åç®æ³æ¬èº«ä¸ä¼æ§è¡å®¹å¨çæä½ï¼å®ä»¬åªä¼è¿è¡äºè¿­ä»£å¨ä¹ä¸ï¼æ§è¡è¿­ä»£å¨çæä½
* å°ç®æ³åå®¹å¨çæåå½æ°åºåå¼æ¥



## 10.3 å®å¶æä½

### 10.3.1 åç®æ³ä¼ éå½æ°

> 10.11ç¼åç¨åºï¼ä½¿ç¨ `stable_sort` å `isShorter` å°ä¼ éç»ä½ ç `elimDups` çæ¬ç `vector` æåºãæå° `vector`çåå®¹ï¼éªè¯ä½ çç¨åºçæ­£ç¡®æ§ã

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate

void print(const std::vector<std::string> &v){
	for(auto i:v)std::cout << i << " ";
	std::cout << std::endl;
}

bool isShorter(const std::string &s1,const std::string &s2){
	return s1.size() < s2.size();
}
void elimDups(std::vector<std::string> &words){
	std::stable_sort(words.begin(),words.end(),isShorter);
	words.erase(std::unique(words.begin(),words.end()),words.end());
}


int main(){
	std::vector<std::string>vs;
	std::string s;
	while(std::cin>>s)vs.push_back(s);
	print(vs);
	elimDups(vs);
	print(vs);
	return 0;
}
```



> 10.12 ç¼ååä¸º `compareIsbn` çå½æ°ï¼æ¯è¾ä¸¤ä¸ª `Sales_data` å¯¹è±¡ç`isbn()` æåãä½¿ç¨è¿ä¸ªå½æ°æåºä¸ä¸ªä¿å­ `Sales_data` å¯¹è±¡ç `vector`ã

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate

class Sales_data;
std::istream &read(std::istream&,Sales_data&);

class Sales_data {

friend std::istream & read(std::istream &,Sales_data &);
friend std::ostream & print(std::ostream &,const Sales_data &);
friend Sales_data add(const Sales_data &,const Sales_data &);

private:
	std::string bookNo;
	unsigned units_sold = 0;
	double revenue = 0.0;
public:	
	Sales_data& combine(const Sales_data &rhs);
	std::string isbn()const{return bookNo;}
	double avg_price()const;
	
	Sales_data(const std::string &s,unsigned u,double p)
		:bookNo(s),units_sold(u),revenue(u*p){}
	Sales_data() : Sales_data("",0,0){}
	Sales_data(const std::string &s):Sales_data(s,0,0){}
	Sales_data(std::istream &is):Sales_data(){read(is,*this);}
};


double Sales_data::avg_price()const{
	return units_sold?(revenue/units_sold):0;
}

Sales_data& Sales_data::combine(const Sales_data &rhs){
	units_sold += rhs.units_sold;
	revenue += rhs.revenue;
	return *this;
}

std::istream & read(std::istream &is,Sales_data &item){
	double price = 0;
	is >> item.bookNo >> item.units_sold >> price;
	item.revenue = item.units_sold * price;
	return is;
}

std::ostream & print(std::ostream &os,const Sales_data &item){
	os << item.isbn() << " "<< item.units_sold << " "
		<< item.revenue << " " << item.avg_price();
	return os;
}

Sales_data add(const Sales_data &lhs,const Sales_data &rhs){
	Sales_data sum = lhs;
	sum.combine(rhs);
	return sum;
}

bool compareIsbn(const Sales_data &A,const Sales_data&B){
	return A.isbn() < B.isbn(); 
}

int main(){
	Sales_data a("aa"),b("bb"),c("cc"),d("aaa");
	std::vector<Sales_data>v{b,a,c,d};

	for(auto i:v){
		print(std::cout,i)<<std::endl;
	}
	std::sort(v.begin(),v.end(),compareIsbn);
	std::cout << std::endl;
	for(auto i:v){
		print(std::cout,i)<<std::endl;
	}
	return 0;
}
```



> 10.13 æ ååºå®ä¹äºåä¸º `partition` çç®æ³ï¼å®æ¥åä¸ä¸ªè°è¯ï¼å¯¹å®¹å¨åå®¹è¿è¡ååï¼ä½¿å¾è°è¯ä¸º`true` çå¼ä¼æå¨å®¹å¨çååé¨åï¼èä½¿å¾è°è¯ä¸º `false` çå¼ä¼æå¨ååé¨åãç®æ³è¿åä¸ä¸ªè¿­ä»£å¨ï¼æåæåä¸ä¸ªä½¿è°è¯ä¸º `true` çåç´ ä¹åçä½ç½®ãç¼åå½æ°ï¼æ¥åä¸ä¸ª `string`ï¼è¿åä¸ä¸ª `bool` å¼ï¼æåº `string` æ¯å¦æ5ä¸ªææ´å¤å­ç¬¦ãä½¿ç¨æ­¤å½æ°åå `words`ãæå°åºé¿åº¦å¤§äºç­äº5çåç´ ã

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>

bool check(const std::string &s){
	return s.size()>5;
}
int main(){
	std::vector<std::string>v={"a","aa","bbbbda","dafwgw","cccccc"};
	auto stop = std::partition(v.begin(), v.end(), check);
	for(auto iter = v.begin(); iter != stop; ++iter){
		std::cout << *iter << std::endl;
	}
	return 0;
}

```



### 10.3.2 lambda è¡¨è¾¾å¼

> 10.14 ç¼åä¸ä¸ª`lambda`ï¼æ¥åä¸¤ä¸ª`int`ï¼è¿åå®ä»¬çåã

```cpp
auto f = [](int i,int j){return i+j;}
```



> 10.15 ç¼åä¸ä¸ª `lambda` ï¼æè·å®æå¨å½æ°ç `int`ï¼å¹¶æ¥åä¸ä¸ª `int`åæ°ã`lambda` åºè¯¥è¿åæè·ç `int` å `int` åæ°çåã

```cpp
int x = 10;
auto f = [x](int i) { i + x; };
```



> 10.16 ä½¿ç¨ `lambda` ç¼åä½ èªå·±çæ¬ç `biggies`ã

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>

void print(const std::vector<std::string> &v){
	for(auto i:v)std::cout << i << " ";
	std::cout << std::endl;
}

void elimDups(std::vector<std::string> &words){
	std::sort(words.begin(),words.end());
	words.erase(std::unique(words.begin(),words.end()),words.end());
}

std::string make_plural(const int count,const std::string word,const std::string &suf){
	return count>1?(word+suf):word;
}
void biggies(std::vector<std::string> &words,std::vector<std::string>::size_type sz){
	elimDups(words);
	std::stable_sort(words.begin(),words.end(),
					[](const std::string &a,const std::string &b){return a.size()<b.size();});
	auto wc = find_if(words.begin(),words.end(),
					[sz](const std::string &a){return a.size()>=sz;});
	auto count = words.end() -wc;
	std::cout << count << " " << make_plural(count,"word","s") 
			<<" of length " << sz << " or longer" << std::endl;
}
int main(){
	std::vector<std::string>v={"123","123","abc","gnoawdnoi","ddddd","ddddd"};
	biggies(v,3);
	return 0;
}
```



> 10.17 éå10.3.1èç»ä¹ 10.12çç¨åºï¼å¨å¯¹`sort`çè°ç¨ä¸­ä½¿ç¨ `lambda` æ¥ä»£æ¿å½æ° `compareIsbn`ã

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate

class Sales_data;
std::istream &read(std::istream&,Sales_data&);

class Sales_data {

friend std::istream & read(std::istream &,Sales_data &);
friend std::ostream & print(std::ostream &,const Sales_data &);
friend Sales_data add(const Sales_data &,const Sales_data &);

private:
	std::string bookNo;
	unsigned units_sold = 0;
	double revenue = 0.0;
public:	
	Sales_data& combine(const Sales_data &rhs);
	std::string isbn()const{return bookNo;}
	double avg_price()const;
	
	Sales_data(const std::string &s,unsigned u,double p)
		:bookNo(s),units_sold(u),revenue(u*p){}
	Sales_data() : Sales_data("",0,0){}
	Sales_data(const std::string &s):Sales_data(s,0,0){}
	Sales_data(std::istream &is):Sales_data(){read(is,*this);}
};


double Sales_data::avg_price()const{
	return units_sold?(revenue/units_sold):0;
}

Sales_data& Sales_data::combine(const Sales_data &rhs){
	units_sold += rhs.units_sold;
	revenue += rhs.revenue;
	return *this;
}

std::istream & read(std::istream &is,Sales_data &item){
	double price = 0;
	is >> item.bookNo >> item.units_sold >> price;
	item.revenue = item.units_sold * price;
	return is;
}

std::ostream & print(std::ostream &os,const Sales_data &item){
	os << item.isbn() << " "<< item.units_sold << " "
		<< item.revenue << " " << item.avg_price();
	return os;
}

Sales_data add(const Sales_data &lhs,const Sales_data &rhs){
	Sales_data sum = lhs;
	sum.combine(rhs);
	return sum;
}

int main(){
	Sales_data a("aa"),b("bb"),c("cc"),d("aaa");
	std::vector<Sales_data>v{b,a,c,d};

	for(auto i:v){
		print(std::cout,i)<<std::endl;
	}
	std::sort(v.begin(),v.end(),
			[](const Sales_data &A,const Sales_data&B){return A.isbn() < B.isbn(); });
	std::cout << std::endl;
	for(auto i:v){
		print(std::cout,i)<<std::endl;
	}
	return 0;
}
```



> 10.18 éå `biggies`ï¼ç¨ `partition` ä»£æ¿ `find_if`ãæä»¬å¨10.3.1èç»ä¹ 10.13ä¸­ä»ç»äº `partition` ç®æ³ã

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>

void elimDups(std::vector<std::string> &words){
	std::sort(words.begin(),words.end());
	words.erase(std::unique(words.begin(),words.end()),words.end());
}

std::string make_plural(const int count,const std::string word,const std::string &suf){
	return count>1?(word+suf):word;
}
void biggies(std::vector<std::string> &words,std::vector<std::string>::size_type sz){
	elimDups(words);

	auto wc = std::partition(words.begin(),words.end(),
							[sz](const std::string &a){return a.size() >= sz;});

	for(auto iter = words.begin(); iter != wc; ++iter){
		std::cout << *iter << std::endl;
	}
	auto count = wc - words.begin();
	std::cout << count << " " << make_plural(count,"word","s") 
			<<" of length " << sz << " or longer" << std::endl;
}
int main(){
	std::vector<std::string>v={"123","123","abc","gnoawdnoi","ddddd","ddddd"};
	biggies(v,5);
	return 0;
}
```



> 10.19 ç¨ `stable_partition` éååä¸é¢çç¨åºï¼ä¸ `stable_sort` ç±»ä¼¼ï¼å¨åååçåºåä¸­ç»´æåæåç´ çé¡ºåºã

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>

void elimDups(std::vector<std::string> &words){
	std::sort(words.begin(),words.end());
	words.erase(std::unique(words.begin(),words.end()),words.end());
}

std::string make_plural(const int count,const std::string word,const std::string &suf){
	return count>1?(word+suf):word;
}
void biggies(std::vector<std::string> &words,std::vector<std::string>::size_type sz){
	elimDups(words);

	auto wc = std::stable_partition(words.begin(),words.end(),
							[sz](const std::string &a){return a.size() >= sz;});

	for(auto iter = words.begin(); iter != wc; ++iter){
		std::cout << *iter << std::endl;
	}
	auto count = wc - words.begin();
	std::cout << count << " " << make_plural(count,"word","s") 
			<<" of length " << sz << " or longer" << std::endl;
}
int main(){
	std::vector<std::string>v={"123","123","abc","gnoawdnoi","ddddd","ddddd"};
	biggies(v,5);
	return 0;
}
```



### 10.3.3 lambda æè·åè¿å

> 10.20 æ ååºå®ä¹äºä¸ä¸ªåä¸º `count_if` çç®æ³ãç±»ä¼¼ `find_if`ï¼æ­¤å½æ°æ¥åä¸å¯¹è¿­ä»£å¨ï¼è¡¨ç¤ºä¸ä¸ªè¾å¥èå´ï¼è¿æ¥åä¸ä¸ªè°è¯ï¼ä¼å¯¹è¾å¥èå´ä¸­æ¯ä¸ªåç´ æ§è¡ã`count_if`è¿åä¸ä¸ªè®¡æ°å¼ï¼è¡¨ç¤ºè°è¯æå¤å°æ¬¡ä¸ºçãä½¿ç¨`count_if`éåæä»¬ç¨åºä¸­ç»è®¡æå¤å°åè¯é¿åº¦è¶è¿6çé¨åã

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>

int main(){
	std::vector<std::string>v={"abc","defghi","jklabcd","helloword"};
	std::cout << count_if(v.begin(), v.end(),
		[](const std::string &a){return a.size()>6;});

	return 0;
}
```



> 10.21 ç¼åä¸ä¸ª `lambda`ï¼æè·ä¸ä¸ªå±é¨ `int` åéï¼å¹¶éååéå¼ï¼ç´è³å®åä¸º0ãä¸æ¦åéåä¸º0ï¼åè°ç¨`lambda`åºè¯¥ä¸åéååéã`lambda`åºè¯¥è¿åä¸ä¸ª`bool`å¼ï¼æåºæè·çåéæ¯å¦ä¸º0ã

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>

int main(){
	int i = 10;
	auto f = [&i]()->bool {
		return (i==0?true:!(i--));
	};
	while(!f())std::cout << i << std::endl;
}
```



### 10.3.4 åæ°ç»å®

> 10.22 éåç»è®¡é¿åº¦å°äºç­äº6çåè¯æ°éçç¨åºï¼ä½¿ç¨å½æ°ä»£æ¿`lambda`ã

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>

bool check(const std::string &a){
	return a.size() <= 6;
}
int main(){
	std::vector<std::string>v={"abc","defghi","jklabcd","helloword"};
	std::cout << count_if(v.begin(), v.end(),check);

	return 0;
}
```



> 10.23 `bind` æ¥åå ä¸ªåæ°ï¼

åè®¾è¢«ç»å®çå½æ°æ¥å `n` ä¸ªåæ°ï¼é£ä¹`bind` æ¥å`n + 1`ä¸ªåæ°ã



> 10.24 ç»å®ä¸ä¸ª`string`ï¼ä½¿ç¨ `bind` å `check_size` å¨ä¸ä¸ª `int` ç`vector` ä¸­æ¥æ¾ç¬¬ä¸ä¸ªå¤§äº`string`é¿åº¦çå¼ã

```cpp
#include <cstddef>
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>
#include <functional>

bool check_size(const std::string &a,std::size_t sz){
	return a.size() < sz;
}
int main(){
	std::vector<int> vec {0,1,2,3,4,5,6,7,8};
	std::string s("abcd");
	auto result = std::find_if(vec.cbegin(),vec.cend(),std::bind(check_size,s,std::placeholders::_1));
	if(result != vec.end())std::cout << *result << std::endl;
	else std::cout << "Not found" << std::endl;
	return 0;
}
```



> 10.25 å¨10.3.2èçç»ä¹ ä¸­ï¼ç¼åäºä¸ä¸ªä½¿ç¨`partition` ç`biggies`çæ¬ãä½¿ç¨ `check_size` å `bind` éåæ­¤å½æ°ã

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>
#include <functional>

void elimDups(std::vector<std::string> &words){
	std::sort(words.begin(),words.end());
	words.erase(std::unique(words.begin(),words.end()),words.end());
}

std::string make_plural(const int count,const std::string word,const std::string &suf){
	return count>1?(word+suf):word;
}

bool check_size(const std::string &a,std::vector<std::string>::size_type sz){
	return a.size() >= sz;
}

void biggies(std::vector<std::string> &words,std::vector<std::string>::size_type sz){
	elimDups(words);

	auto wc = std::stable_partition(words.begin(),words.end(),
									bind(check_size,std::placeholders::_1,sz));

	for(auto iter = words.begin(); iter != wc; ++iter){
		std::cout << *iter << std::endl;
	}
	auto count = wc - words.begin();
	std::cout << count << " " << make_plural(count,"word","s") 
			<<" of length " << sz << " or longer" << std::endl;
}
int main(){
	std::vector<std::string>v={"123","123","abc","gnoawdnoi","ddddd","ddddd"};
	biggies(v,5);
	return 0;
}
```



## 10.4  åæ¢è¿­ä»£å¨

### 10.4.1 æå¥è¿­ä»£å¨

> 10.26è§£éä¸ç§æå¥è¿­ä»£å¨çä¸åä¹å¤ã

* `back_inserter`åå»ºä¸ä¸ªä½¿ç¨`push_back`çè¿­ä»£å¨
* `front_inserter`åå»ºä¸ä¸ªä½¿ç¨`push_front`çè¿­ä»£å¨å¨
* `inserter`åå»ºä¸ä¸ªä½¿ç¨`insert`çè¿­ä»£å¨



> 10.27 é¤äº `unique` ä¹å¤ï¼æ ååºè¿å®ä¹äºåä¸º `unique_copy` çå½æ°ï¼å®æ¥åç¬¬ä¸ä¸ªè¿­ä»£å¨ï¼è¡¨ç¤ºæ·è´ä¸éå¤åç´ çç®çä½ç½®ãç¼åä¸ä¸ªç¨åºï¼ä½¿ç¨ `unique_copy`å°ä¸ä¸ª`vector`ä¸­ä¸éå¤çåç´ æ·è´å°ä¸ä¸ªåå§åä¸ºç©ºç`list`ä¸­ã

```cpp
#include <iostream>
#include <algorithm>
#include <iterator>
#include <string>
#include <list>
#include <vector>
#include <numeric>
#include <functional>

int main(){
	std::vector<int>vec={0,0,1,1,2,3,4,4};
	std::sort(vec.begin(),vec.end());
	
	std::list<int>ls;
	std::unique_copy(vec.begin(),vec.end(),std::back_inserter(ls));

	for(auto i:ls)std::cout << i << " ";
	return 0;
}
```



> 10.28 ä¸ä¸ª`vector` ä¸­ä¿å­ 1 å° 9ï¼å°å¶æ·è´å°ä¸ä¸ªå¶ä»å®¹å¨ä¸­ãåå«ä½¿ç¨`inserter`ã`back_inserter` å `front_inserter` å°åç´ æ·»å å°ä¸ä¸ªå®¹å¨ä¸­ãå¯¹æ¯ç§ `inserter`ï¼ä¼°è®¡è¾åºåºåæ¯ææ ·çï¼è¿è¡ç¨åºéªè¯ä½ çä¼°è®¡æ¯å¦æ­£ç¡®ã

```cpp
#include <iostream>
#include <algorithm>
#include <iterator>
#include <string>
#include <list>
#include <vector>
#include <numeric>
#include <functional>

void print(const std::list<int> &ls){
	for(auto i:ls)std::cout << i << " ";
	std::cout << std::endl;
}
int main(){
	std::vector<int>vec={1,2,3,4,5,6,7,8,9};
	std::sort(vec.begin(),vec.end());
	
	std::list<int>ls1,ls2,ls3;
	std::copy(vec.cbegin(),vec.cend(),std::inserter(ls1,ls1.begin()));
	std::copy(vec.cbegin(),vec.cend(),std::front_inserter(ls2));
	std::copy(vec.cbegin(),vec.cend(),std::back_inserter(ls3));

	print(ls1);
	print(ls2);
	print(ls3);
	
	return 0;
}
```



### 10.4.2 iostream è¿­ä»£å¨

> 10.29 ç¼åç¨åºï¼ä½¿ç¨æµè¿­ä»£å¨è¯»åä¸ä¸ªææ¬æä»¶ï¼å­å¥ä¸ä¸ª`vector`ä¸­ç`string`éã

```cpp
#include <iostream>
#include <algorithm>
#include <iterator>
#include <string>
#include <list>
#include <vector>
#include <numeric>
#include <functional>
#include <fstream>

int main(int argc,char **argv){
	std::vector<std::string>vs;
	std::ifstream ifs(argv[1]);
	std::istream_iterator<std::string>ifs_iter(ifs),eof;
	std::copy(ifs_iter,eof,std::back_inserter(vs));
	std::copy(vs.cbegin(),vs.cend(),std::ostream_iterator<std::string>(std::cout,"\n"));
	return 0;
}
```



> 10.30 ä½¿ç¨æµè¿­ä»£å¨ã`sort` å `copy` ä»æ åè¾å¥è¯»åä¸ä¸ªæ´æ°åºåï¼å°å¶æåºï¼å¹¶å°ç»æåå°æ åè¾åºã

```cpp
#include <iostream>
#include <algorithm>
#include <iterator>
#include <string>
#include <list>
#include <vector>
#include <numeric>
#include <functional>
#include <fstream>

int main(int argc,char **argv){
	std::vector<std::string>v;
	std::istream_iterator<std::string>ifs_iter(std::cin),eof;
	
	std::copy(ifs_iter,eof,std::back_inserter(v));
	std::sort(v.begin(),v.end());

	std::copy(v.cbegin(),v.cend(),std::ostream_iterator<std::string>(std::cout," "));
	return 0;
}
```



> 10.31 ä¿®æ¹åä¸é¢çç¨åºï¼ä½¿å¶åªæå°ä¸éå¤çåç´ ãä½ çç¨åºåºè¯¥ä½¿ç¨ `unique_copy`ã

```cpp
#include <iostream>
#include <algorithm>
#include <iterator>
#include <string>
#include <list>
#include <vector>
#include <numeric>
#include <functional>
#include <fstream>

int main(int argc,char **argv){
	std::istream_iterator<int> in_iter(std::cin);
	std::istream_iterator<int> eof;
	std::ostream_iterator<int> out_iter(std::cout," ");

	std::vector<int>ivec;

	while(in_iter!=eof){
		ivec.push_back(*in_iter++);
	}
	std::sort(ivec.begin(),ivec.end());
	std::unique_copy(ivec.cbegin(),ivec.cend(),out_iter);


	return 0;
}
```



> 10.32 éå1.6èä¸­çä¹¦åºç¨åºï¼ä½¿ç¨ä¸ä¸ª`vector`ä¿å­äº¤æè®°å½ï¼ä½¿ç¨ä¸åç®æ³å®æå¤çãä½¿ç¨ `sort` å10.3.1èä¸­ç `compareIsbn` å½æ°æ¥æåºäº¤æè®°å½ï¼ç¶åä½¿ç¨ `find` å `accumulate` æ±åã

```cpp
#include <iostream>
#include <algorithm>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate
#include <functional>
#include <iterator>
#include "Sales_item.h"//çç¬¬ä¸ç« 1.5.1ç»ä¹ é¢è§£ï¼æå¤´æä»¶çä»£ç 

int main(){
	std::istream_iterator<Sales_item>in_iter(std::cin),eof;
	std::vector<Sales_item>vec;

	while(in_iter != eof){
		vec.push_back(*in_iter++);
	}

	sort(vec.begin(),vec.end(),compareIsbn);
	for(auto beg = vec.cbegin(),end = beg; beg != vec.cend();beg = end){
		end = find_if(beg,vec.cend(),
					[beg](const Sales_item &item){return item.isbn() != beg->isbn();});
		std::cout << std::accumulate(beg,end,Sales_item(beg->isbn())) << std::endl;
	}
	return 0;
}
```



> 10.33 ç¼åç¨åºï¼æ¥åä¸ä¸ªåæ°ï¼ä¸ä¸ªè¾å¥æä»¶åä¸¤ä¸ªè¾åºæä»¶çæä»¶åãè¾å¥æä»¶ä¿å­çåºè¯¥æ¯æ´æ°ãä½¿ç¨ `istream_iterator` è¯»åè¾å¥æä»¶ãä½¿ç¨ `ostream_iterator` å°å¥æ°åå¥ç¬¬ä¸ä¸ªè¾å¥æä»¶ï¼æ¯ä¸ªå¼åé¢é½è·ä¸ä¸ªç©ºæ ¼ãå°å¶æ°åå¥ç¬¬äºä¸ªè¾åºæä»¶ï¼æ¯ä¸ªå¼é½ç¬å ä¸è¡ã

```cpp
#include <iostream>
#include <algorithm>
#include <istream>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate
#include <functional>
#include <iterator>
#include <fstream>

int main(int argc,char** argv){
	if(argc != 4)return -1;
	
	std::ifstream ifs(argv[1]);
	std::ofstream  ofs_odd(argv[2]),ofs_even(argv[3]);

	std::istream_iterator<int> in_iter(ifs),eof;
	std::ostream_iterator<int> out_odd(ofs_odd," "),out_even(ofs_even,"\n");
	
	std::vector<int>odd,even;

	while(in_iter != eof){
		if((*in_iter)%2==0){
			even.push_back(*in_iter);
		}else{
			odd.push_back(*in_iter);
		}
		++in_iter;
	}

	std::copy(odd.cbegin(),odd.cend(),out_odd);
	std::copy(even.cbegin(),even.cend(),out_even);

	return 0;
}
```



### 10.4.3 ååè¿­ä»£å¨

> 10.34 ä½¿ç¨ `reverse_iterator` éåºæå°ä¸ä¸ª`vector`ã

```cpp
#include <iostream>
#include <algorithm>
#include <istream>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate
#include <functional>
#include <iterator>
#include <fstream>

int main(){
	std::vector<int>v={1,2,3,4,5,6};
	for(auto it = v.crbegin(); it != v.crend(); ++it){
		std::cout << *it << " ";
	}
	return 0;
}
```



> 10.35 ä½¿ç¨æ®éè¿­ä»£å¨éåºæå°ä¸ä¸ª`vector`ã

```cpp
#include <iostream>
#include <algorithm>
#include <istream>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate
#include <functional>
#include <iterator>
#include <fstream>

int main(){
	std::vector<int>v={1,2,3,4,5,6};
	for(auto it = v.cend()-1; it != v.cbegin()-1; --it){
		std::cout << *it << " ";
	}
	return 0;
}
```



> 10.36 ä½¿ç¨ `find` å¨ä¸ä¸ª `int` ç`list` ä¸­æ¥æ¾æåä¸ä¸ªå¼ä¸º0çåç´ ã

```cpp
#include <iostream>
#include <algorithm>
#include <istream>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate
#include <functional>
#include <iterator>
#include <fstream>

int main(){
	std::list<int>ls={0,1,2,3,4,5,0,7};
	auto it = std::find(ls.crbegin(),ls.crend(),0);
	std::cout << distance(it,ls.crend());
	return 0;
}
```



> 10.37 ç»å®ä¸ä¸ªåå«10 ä¸ªåç´ ç`vector`ï¼å°ä½ç½®3å°7ä¹é´çåç´ æéåºæ·è´å°ä¸ä¸ª`list`ä¸­ã

```cpp
#include <iostream>
#include <algorithm>
#include <istream>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate
#include <functional>
#include <iterator>
#include <fstream>

int main(){
	std::list<int>ls;
	std::vector<int>vec={0,1,2,3,4,5,6,7,8,9};
	std::copy(vec.crbegin()+2,vec.crbegin()+7,std::back_inserter(ls));

	for(auto i:ls)std::cout << i << " ";
	return 0;
}
```



### 10.5.1 5ç±»è¿­ä»£å¨

> 10.38 ååº5ä¸ªè¿­ä»£å¨ç±»å«ï¼ä»¥åæ¯ç±»è¿­ä»£å¨ææ¯æçæä½ã

* è¾å¥è¿­ä»£å¨å¿é¡»æ¯æï¼`==`,`!=`,`++`,`*`,`->`
* è¾åºè¿­ä»£å¨å¿é¡»æ¯æ(`ostream_iterator`ç±»åä¹æ¯è¾åºè¿­ä»£å¨)ï¼`++`,`*`
* ååè¿­ä»£å¨
	* `==`,`!=`,`++`,`*`,`->`
	* `replace`ç®æ³è¦æ±ååè¿­ä»£å¨ï¼`foward_list`ä¸çè¿­ä»£å¨ä¹æ¯ååè¿­ä»£å¨
* ååè¿­ä»£å¨
	*   `==`,`!=`,`++`,`--`,`*`,`->`
	* ç®æ³`reverser`è¦æ±ååè¿­ä»£å¨ï¼é¤äº`foward_list`å¶ä»æ ååºé½æä¾ååè¿­ä»£å¨
* éæºè®¿é®è¿­ä»£å¨
	*  `==`,`!=`,`<`,`<=`,`>`,`>=`,`++`,`--`,`+`,`+=`,`-`,`-=`,`*`,`->`,`iter[n]`==`*(iter[n])`
	* `sort`è¦æ±éæºè®¿é®è¿ç®ç¬¦ï¼`array`ã`deque`ã`string`å`vector`è¿­ä»£å¨é½æ¯éæºè®¿é®è¿­ä»£å¨ï¼ç¨äºè®¿é®åç½®æ°ç»çæéä¹æ¯



> 10.39 `list` ä¸çè¿­ä»£å¨å±äºåªç±»ï¼`vector`å¢ï¼

`list`ä¸çè¿­ä»£å¨æ¯ååè¿­ä»£å¨

`vector`æ¯éæºè®¿é®è¿­ä»£å¨



> 10.40 ä½ è®¤ä¸º `copy` è¦æ±åªç±»è¿­ä»£å¨ï¼`reverse` å `unique` å¢ï¼

`copy`è¦ä¸¤ä¸ªè¾å¥è¿­ä»£å¨åä¸ä¸ªè¾åºè¿­ä»£å¨

`reverser`è¦æ±ååè¿­ä»£å¨

`unique`è¦æ±éæºè®¿é®è¿­ä»£å¨

 

### 10.5.3 ç®æ³å½åè§è

> 10.41 ä»æ ¹æ®ç®æ³ååæ°çåå­ï¼æè¿°ä¸é¢æ¯ä¸ªæ ååºç®æ³æ§è¡ä»ä¹æä½ï¼

```cpp
replace(beg, end, old_val, new_val);//è¿­ä»£å¨è¿ååçå¼ç¨new_valæ¿æ¢
replace_if(beg, end, pred, new_val);//æ»¡è¶³ pred è°è¯æ¡ä»¶ææ¿æ¢
replace_copy(beg, end, dest, old_val, new_val);//å¤å¶è¿­ä»£å¨èå´åçåç´ å°ç®æ è¿­ä»£å¨ä½ç½®ï¼å¦æåç´ ç­äºæä¸ªæ§å¼ï¼åç¨æ°å¼æ¿æ¢
replace_copy_if(beg, end, dest, pred, new_val);//å¤å¶è¿­ä»£å¨èå´åçåç´ å°ç®æ è¿­ä»£å¨ä½ç½®ï¼æ»¡è¶³è°è¯æ¡ä»¶çåç´ ç¨æ°å¼æ¿æ¢
```



## 10.6 ç¹å®å®¹å¨çç®æ³

> 10.42 ä½¿ç¨ `list` ä»£æ¿ `vector` éæ°å®ç°10.2.3èä¸­çå»é¤éå¤åè¯çç¨åºã

```cpp
#include <iostream>
#include <algorithm>
#include <istream>
#include <string>
#include <list>
#include <vector>
#include <numeric>//accumulate
#include <functional>
#include <iterator>
#include <fstream>

void elimDups(std::list<std::string> &words){
	words.sort();
	words.unique();
}

int main(){
	std::list<std::string> l = { "aa", "aa", "aa", "aa", "aasss", "aa" };
	
	elimDups(l);

	for(const auto& e : l)std::cout << e << " ";

	return 0;
}
```

