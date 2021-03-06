<h1 align="center">ð C++ Primer 0x09 ç»ä¹ é¢è§£</h1>

## 9.1 é¡ºåºå®¹å¨æ¦è¿°

> 9.1 å¯¹äºä¸é¢çç¨åºä»»å¡ï¼`vector`ã`deque`å`list`åªç§å®¹å¨æä¸ºéåï¼è§£éä½ çéæ©ççç±ãå¦ææ²¡æåªä¸ç§å®¹å¨ä¼äºå¶ä»å®¹å¨ï¼ä¹è¯·è§£éçç±ã
>
> (a) è¯»ååºå®æ°éçåè¯ï¼å°å®ä»¬æå­å¸åºæå¥å°å®¹å¨ä¸­ãæä»¬å°å¨ä¸ä¸ç« ä¸­çå°ï¼å³èå®¹å¨æ´éåè¿ä¸ªé®é¢ã
>
> (b) è¯»åæªç¥æ°éçåè¯ï¼æ»æ¯å°åè¯æå¥å°æ«å°¾ãå é¤æä½å¨å¤´é¨è¿è¡ã
>
> (c) ä»ä¸ä¸ªæä»¶è¯»åæªç¥æ°éçæ´æ°ãå°è¿äºæ°æåºï¼ç¶åå°å®ä»¬æå°å°æ åè¾åºã

* (a)`list`æå¥ä½ç½®ä»»æï¼ä¸ä¸å®å¨é¦å°¾
* (b)`deque`æå¥å é¤æä½å¨å¤´å°¾
* (c)`vector`ä¸éè¦æå¥å é¤ï¼èä¸è¦æåº



## 9.2 å®¹å¨åºæ¦è§

> 9.2 å®ä¹ä¸ä¸ª`list`å¯¹è±¡ï¼å¶åç´ ç±»åæ¯`int`ç`deque`ã

```cpp
std::list<std::deque<int>> ls;
```



### 9.2.1 è¿­ä»£å¨

> 9.3 ææè¿­ä»£å¨èå´çè¿­ä»£å¨æä½éå¶ï¼

* æååä¸ä¸ªå®¹å¨ä¸­çåç´ æèæ¯å®¹å¨åç´ æåä¸ä¸ªåç´ çä½ç½®

* end ä¸å¨ beginä¹å



> 9.4 ç¼åå½æ°ï¼æ¥åä¸å¯¹æå`vector<int>`çè¿­ä»£å¨åä¸ä¸ª`int`å¼ãå¨ä¸¤ä¸ªè¿­ä»£å¨æå®çèå´ä¸­æ¥æ¾ç»å®çå¼ï¼è¿åä¸ä¸ªå¸å°å¼æ¥æåºæ¯å¦æ¾å°ã

```cpp
#include <vector>
#include <iostream>

using  Iter = std::vector<int>::iterator;

bool find(Iter beg, Iter end, int x){
	while(beg!=end){
		if((*beg)==x)return true;
		++beg;
	}
	return false;
}

int main(){
	std::vector<int>vec={1,2,3,4,5,6,7};
	if(find(vec.begin(),vec.end(),3)){
		std::cout << "yes" << std::endl;
	}else{
		std::cout << "no" << std::endl;
	}
	return 0;
}
```



> 9.5 éåä¸ä¸é¢çå½æ°ï¼è¿åä¸ä¸ªè¿­ä»£å¨æåæ¾å°çåç´ ãæ³¨æï¼ç¨åºå¿é¡»å¤çæªæ¾å°ç»å®å¼çæåµã

```cpp
#include <vector>
#include <iostream>

using  Iter = std::vector<int>::iterator;

Iter find(Iter beg, Iter end, int x){
	while(beg!=end){
		if((*beg)==x)break;
		++beg;
	}
	return beg;
}

int main(){
	std::vector<int>vec={1,2,3,4,5,6,7};
	Iter ret = find(vec.begin(),vec.end(),3);
	if(ret == vec.end()){
		std::cout << "no" << std::endl;
	}else{
		std::cout << *ret << " " << ret-vec.begin() << std::endl;
	}
	return 0;
}
```



> 9.6 ä¸é¢çç¨åºæä½éè¯¯ï¼ä½ åºè¯¥å¦ä½ä¿®æ¹å®ï¼

```cpp
list<int> lst1;
list<int>::iterator iter1 = lst1.begin(),
					iter2 = lst1.end();
while (iter1 < iter2) /* ... */
```

ä¸ `vector` å `deque` ä¸åï¼`list` çè¿­ä»£å¨ä¸æ¯æ `<` è¿ç®ï¼åªæ¯æéå¢ãéåã`=` ä»¥å `!=` è¿ç®ã

```cpp
while(iter1 != iter2)
```



### 9.2.2 å®¹å¨ç±»æå

> 9.7 ä¸ºäºç´¢å¼`int`ç`vector`ä¸­çåç´ ï¼åºè¯¥ä½¿ç¨ä»ä¹ç±»åï¼

`std::vector<int>::size_type`



> 9.8 ä¸ºäºè¯»å`string`ç`list`ä¸­çåç´ ï¼åºè¯¥ä½¿ç¨ä»ä¹ç±»åï¼å¦æåå¥`list`ï¼ååºè¯¥ä½¿ç¨ä»ä¹ç±»åï¼

è¯»`std::list<string>::const_iterator`

å`std::list<string>::iterator`



### 9.2.3 begin å end æå

> 9.9 `begin`å`cbegin`ä¸¤ä¸ªå½æ°æä»ä¹ä¸åï¼

`begin`è¿åæ®éè¿­ä»£å¨

`cbegin`è¿åå¸¸éè¿­ä»£å¨



> 9.10 ä¸é¢4ä¸ªå¯¹è±¡åå«æ¯ä»ä¹ç±»åï¼

```cpp
vector<int> v1;
const vector<int> v2;
auto it1 = v1.begin(), it2 = v2.begin();
auto it3 = v1.cbegin(), it4 = v2.cbegin();
```

* it1åit2`vector<int>::iterator`
* it3åit4`vector<int>::const_iterator`



### 9.2.4 å®¹å¨å®ä¹ååå§å

> 9.11 å¯¹6ç§åå»ºååå§å`vector`å¯¹è±¡çæ¹æ³ï¼æ¯ä¸ç§é½ç»åºä¸ä¸ªå®ä¾ãè§£éæ¯ä¸ª`vector`åå«ä»ä¹å¼ã

```cpp
vector<int> vec1;//ç©º
vector<int> vec2(10);//10ä¸ª0
vector<int> vec3(10,1);//10ä¸ª1
vector<int> vec4{1,2,3,4,5,6,7,8,9,10};//1ï½10
vector<int> vec5(vec4);//1ï½10
vector<int> vec6(vec5.begin(),vec5.end());//1ï½10
```



> 9.12 å¯¹äºæ¥åä¸ä¸ªå®¹å¨åå»ºå¶æ·è´çæé å½æ°ï¼åæ¥åä¸¤ä¸ªè¿­ä»£å¨åå»ºæ·è´çæé å½æ°ï¼è§£éå®ä»¬çä¸åã

* å°ä¸ä¸ªå®¹å¨åå»ºä¸ºå¦ä¸ä¸ªå®¹å¨çæ·è´æ¹æ³æä¸¤ç§

	* ç´æ¥æ·è´æ´ä¸ªå®¹å¨ï¼ä¸¤ä¸ªå®¹å¨çç±»åååç´ ç±»åå¿é¡»å¹é
	* æ·è´ç±ä¸ä¸ªè¿­ä»£å¨å¯¹æå®çåç´ èå´ï¼`array`é¤å¤ï¼ï¼ä¸è¦æ±å®¹å¨ç±»åç¸åï¼åç´ ç±»åä¹å¯ä»¥ä¸åï¼åªè¦è½å°æ·è´çåç´ è½¬æ¢



> 9.13 å¦ä½ä»ä¸ä¸ª`list<int>`åå§åä¸ä¸ª`vector<double>`ï¼ä»ä¸ä¸ª`vector<int>`åè¯¥å¦ä½åå»ºï¼ç¼åä»£ç éªè¯ä½ çç­æ¡ã

```cpp
#include <vector>
#include <iostream>
#include <list>
using namespace std;

int main(){
	list<int>ls{1,2,3,4};
	vector<int>vec{5,6,7,8};
	
	vector<double>dvec1(ls.begin(),ls.end());
	vector<double>dvec2(vec.begin(),vec.end());
	
	for(auto v:dvec1)cout << v << " ";
	cout << endl;
	
	for(auto v:dvec2)cout << v << " ";
	cout << endl;
	return 0;
}
```



### 9.2.5 èµå¼å swap

> 9.14 ç¼åç¨åºï¼å°ä¸ä¸ª`list`ä¸­ç`char *`æéåç´ èµå¼ç»ä¸ä¸ª`vector`ä¸­ç`string`ã

```cpp
list<char*>ls{"abc","def","ghi"};
vector<string>vec;
vec.assign(ls.cbegin(),ls.cend());
```



### 9.2.7 å³ç³»è¿ç®ç¬¦

> 9.15 ç¼åç¨åºï¼å¤å®ä¸¤ä¸ª`vector<int>`æ¯å¦ç¸ç­ã

```cpp
#include <vector>
#include <iostream>
#include <list>
using namespace std;

int main(){
	vector<int>vec1{1,2,3,4,5};
	vector<int>vec2{1,2,3,4,5};
	vector<int>vec3{1,2,3,5,4};
	
	cout << (vec1==vec2?"yes":"no") << endl;
	cout << (vec1==vec3?"yes":"no") << endl;
	return 0;
}
```



> 9.16 éåä¸ä¸é¢çç¨åºï¼æ¯è¾ä¸ä¸ªlistä¸­çåç´ åä¸ä¸ªvectorä¸­çåç´ ã

```cpp
#include <vector>
#include <iostream>
#include <list>
using namespace std;

int main(){
	vector<int>vec1{1,2,3,4,5};
	list<int>ls1{1,2,3,4,5};
	list<int>ls2{1,2,3,5,4};
	
	cout << (vec1==vector<int>(ls1.begin(),ls1.end())?"yes":"no") << endl;
	cout << (vec1==vector<int>(ls2.begin(),ls2.end())?"yes":"no") << endl;
	return 0;
}
```



> 9.17 åå®`c1`å`c2`æ¯ä¸¤ä¸ªå®¹å¨ï¼ä¸é¢çæ¯è¾æä½æä½éå¶ï¼(**å¦ææçè¯**)

```cpp
if (c1 < c2)
```

* åªæå½å¶ä¸­åç´ ç±»åä¹å®ä¹äºç¸åºçæ¯è¾è¿ç®ç¬¦æ¶ï¼æä»¬æå¯ä»¥ä½¿ç¨å³ç³»è¿ç®ç¬¦æ¥æ¯è¾ä¸¤ä¸ªå®¹å¨
* å³ç³»è¿ç®ç¬¦ä¸¤ä¾§è¿ç®å¯¹è±¡å¿é¡»æ¯ç±»åç¸åçå®¹å¨ä¸å¿é¡»ä¿å­ç¸åç±»åçåç´ 



## 9.3 é¡ºåºå®¹å¨æä½

### 9.3.1 åé¡ºåºå®¹å¨æ·»å åç´ 

> 9.18 ç¼åç¨åºï¼ä»æ åè¾å¥è¯»å`string`åºåï¼å­å¥ä¸ä¸ª`deque`ä¸­ãç¼åä¸ä¸ªå¾ªç¯ï¼ç¨è¿­ä»£å¨æå°`deque`ä¸­çåç´ ã

```cpp
#include <iostream>
#include <string>
#include <deque>
using namespace std;

int main(){
	deque<string>dq;
	string s;
	while(cin>>s)dq.push_back(s);
	for(auto iter=dq.cbegin();iter!=dq.cend();++iter){
		cout << *iter << " ";
	}
	return 0;
}
```



> 9.19 éåä¸ä¸é¢çç¨åºï¼ç¨`list`æ¿ä»£`deque`ãååºç¨åºè¦ååºåªäºæ¹åã

```cpp
#include <iostream>
#include <list>
#include <string>
using namespace std;

int main(){
	list<string>ls;
	string s;
	while(cin>>s)ls.push_back(s);
	for(auto iter=ls.cbegin();iter!=ls.cend();++iter){
		cout << *iter << " ";
	}
	return 0;
}
```



> 9.20 ç¼åç¨åºï¼ä»ä¸ä¸ª`list<int>`æ·è´åç´ å°ä¸¤ä¸ª`deque`ä¸­ãå¼ä¸ºå¶æ°çææåç´ é½æ·è´å°ä¸ä¸ª`deque`ä¸­ï¼èå¥æ°å¼åç´ é½æ·è´å°å¦ä¸ä¸ª`deque`ä¸­ã

```cpp
#include <iostream>
#include <list>
#include <deque>
using namespace std;

int main(){
	list<int>ls={1,2,3,4,5,6,7,8};
	deque<int>odd,even;
	for(auto e:ls){
		if(e&1)odd.push_back(e);
		else even.push_back(e);
	}
	
	for(auto e:odd)cout << e << " ";
	cout << endl;
	for(auto e:even)cout << e << " ";
	
	return 0;
}
```



> 9.21 å¦ææä»¬å°ç¬¬308é¡µä¸­ä½¿ç¨`insert`è¿åå¼å°åç´ æ·»å å°`list`ä¸­çå¾ªç¯ç¨åºæ¹åä¸ºå°åç´ æå¥å°`vector`ä¸­ï¼åæå¾ªç¯å°å¦ä½å·¥ä½ã

æ²¡å¥åºå«

`insert`è¿åå¼æåæå¥çæ°åç´ ï¼å¯ä»¥åå©`insert`å®ç°`push_front`ç¸åçåè½



> 9.22 åå®`iv`æ¯ä¸ä¸ª`int`ç`vector`ï¼ä¸é¢çç¨åºå­å¨ä»ä¹éè¯¯ï¼ä½ å°å¦ä½ä¿®æ¹ï¼

```cpp
vector<int>::iterator iter = iv.begin(),
					  mid = iv.begin() + iv.size() / 2;
while (iter != mid)
	if (*iter == some_val)
		iv.insert(iter, 2 * some_val);
```

* å¾ªç¯éè¿­ä»£å¨æ²¡åï¼æ­»å¾ªç¯
* å¾ªç¯éæå¥æä½ï¼è¿­ä»£å¨æ²¡ææ´æ°ï¼è¿­ä»£å¨ä¼å¤±æ

å ä¸ªè¦ç¹

æå¥å`mid`ä¼å¤±æï¼è¦æ´æ°ï¼ä¿è¯è¿æ¯åå§ä¸­ç¹ç»æ­¢

æå¥åç´ åè¦å¤ç§»å¨ä¸æ­¥

è¿­ä»£å¨ä¹ä¼å¤±æï¼è¦æ´æ°

```cpp
#include <iostream>
#include <list>
#include <vector>
using namespace std;

int main(){
	vector<int>iv={5,4,4,4,3,1};
	int some_value = 4;
	int move = 0;
	vector<int>::iterator iter = iv.begin();
	int siz=iv.size();
	while(iter != (iv.begin() + siz/2+move)){
		if(*iter == some_value){
			iter = iv.insert(iter,some_value*2);
			move++;
			iter++;iter++;
		}
		else ++iter;
	}
	for(auto v:iv)cout << v << " ";
		
	return 0;
}
```

### 9.3.2 è®¿é®åç´ 

> 9.23 å¨æ¬èç¬¬ä¸ä¸ªç¨åºä¸­ï¼è¥`c.size()` ä¸º1ï¼å`val`ã`val2`ã`val3`å`val4`çå¼ä¼æ¯ä»ä¹ï¼

é½æ¯å®¹å¨éå¯ä¸çé£ä¸ªåç´ çå¼



> 9.24 ç¼åç¨åºï¼åå«ä½¿ç¨`at`ãä¸æ è¿ç®ç¬¦ã`front` å `begin` æåä¸ä¸ª`vector`ä¸­çç¬¬ä¸ä¸ªåç´ ãå¨ä¸ä¸ªç©º`vector`ä¸æµè¯ä½ çç¨åºã

```cpp
#include <iostream>
#include <vector>

int main(){
    std::vector<int> vec;
    
    std::cout << vec.at(0);
    //terminate called after throwing an instance of 'std::out_of_range       
    std::cout << vec[0];//æ®µéè¯¯        
    std::cout << vec.front();//æ®µéè¯¯    
    std::cout << *vec.begin();//æ®µéè¯¯  
      
    return 0;
}
```



### 9.3.3 å é¤åç´ 

> 9.25 å¯¹äºç¬¬312é¡µä¸­å é¤ä¸ä¸ªèå´åçåç´ çç¨åºï¼å¦æ `elem1` ä¸ `elem2` ç¸ç­ä¼åçä»ä¹ï¼å¦æ `elem2` æ¯å°¾åè¿­ä»£å¨ï¼æè `elem1` å `elem2` çä¸ºå°¾åè¿­ä»£å¨ï¼åä¼åçä»ä¹ï¼

* å¦æ `elem1` ä¸ `elem2` ç¸ç­ï¼è¿­ä»£å¨èå´ä¸ºç©ºï¼ä»ä¹é½ä¸å
* å¦æ `elem2` æ¯å°¾åè¿­ä»£å¨ï¼ä»`elem1`å¼å§å å°æ«å°¾
* `elem1` å `elem2` çä¸ºå°¾åè¿­ä»£å¨ï¼ä»ä¹é½ä¸å



> 9.26 ä½¿ç¨ä¸é¢ä»£ç å®ä¹ç`ia`ï¼å°`ia`æ·è´å°ä¸ä¸ª`vector`åä¸ä¸ª`list`ä¸­ãæ¯ç¨åè¿­ä»£å¨çæ¬ç`erase`ä»`list`ä¸­å é¤å¥æ°åç´ ï¼ä»`vector`ä¸­å é¤å¶æ°åç´ ã

```cpp
int ia[] = { 0, 1, 1, 2, 3, 5, 8, 13, 21, 55, 89 };
```



```cpp
#include <iostream>
#include <vector>
#include <list>

int main(){
	int ia[] = { 0, 1, 1, 2, 3, 5, 8, 13, 21, 55, 89 };
    
    std::vector<int>vec;
    std::list<int>ls;
    
    for(auto i:ia)vec.push_back(i);
    for(auto i:ia)ls.push_back(i);

	auto it1=vec.begin();
	auto it2=ls.begin();
	
	while(it1!=vec.end()){
		if((1&(*it1))==0){
			it1=vec.erase(it1);
		}else{
			++it1;
		}
	}

	
	while(it2!=ls.end()){
		if(1&(*it2)){
			it2=ls.erase(it2);
		}else{
			++it2;
		}
	}
	

	for(auto i:vec)std::cout << i << " ";
	std::cout << std::endl;
	for(auto i:ls)std::cout << i << " ";
	
    return 0;
}
```

###  9.3.4 ç¹æ®ç forward_list  æä½

> 9.27 ç¼åç¨åºï¼æ¥æ¾å¹¶å é¤`forward_list<int>`ä¸­çå¥æ°åç´ ã

```cpp
#include <iostream>
#include <forward_list>

int main(){
	std::forward_list<int>flst{1,2,3,4,5};
	auto prev = flst.before_begin();
	auto curr = flst.begin();
	while(curr != flst.end()){
		if(*curr%2)
			curr = flst.erase_after(prev);
		else{
			prev = curr;
			++curr;
		}
	}
	for(auto i:flst)std::cout << i << " ";
    return 0;
}
```



> 9.28 ç¼åå½æ°ï¼æ¥åä¸ä¸ª`forward_list<string>`åä¸¤ä¸ª`string`å±ä¸ä¸ªåæ°ãå½æ°åºå¨é¾è¡¨ä¸­æ¥æ¾ç¬¬ä¸ä¸ª`string`ï¼å¹¶å°ç¬¬äºä¸ª`string`æå¥å°ç´§æ¥çç¬¬ä¸ä¸ª`string`ä¹åçä½ç½®ãè¥ç¬¬ä¸ä¸ª`string`æªå¨é¾è¡¨ä¸­ï¼åå°ç¬¬äºä¸ª`string`æå¥å°é¾è¡¨æ«å°¾ã

```cpp
#include <iostream>
#include <forward_list>

using std::string;
using FL = std::forward_list<string>;

void fun(FL& flst,string a,string b){
	auto prev = flst.before_begin();
	auto curr = flst.begin();	
	while(curr != flst.end()){
		if(*curr == a){
			flst.insert_after(curr,b);
			return;
		}
		prev = curr;
		++curr;
	}
	return;
}

int main(){
	FL flst={"abc","def","ghi"};
	fun(flst,"abc","ABC");
	for(auto i:flst)std::cout << i << " ";
    return 0;
}
```

### 9.3.5 æ¹åå®¹å¨å¤§å°

> 9.29 åå®`vec`åå«25ä¸ªåç´ ï¼é£ä¹`vec.resize(100)`ä¼åä»ä¹ï¼å¦ææ¥ä¸æ¥è°ç¨`vec.resize(10)`ä¼åä»ä¹ï¼

`vec.resize(100)`æ«å°¾å 75ä¸ª0(å¦ææ¯int)

`vec.resize(10)`å é¤æ«å°¾90ä¸ªåç´ 



> 9.30 æ¥ååä¸ªåæ°ç`resize`çæ¬å¯¹åç´ ç±»åæä»ä¹éå¶ï¼å¦ææçè¯ï¼ï¼

åç´ ç±»åå¿é¡»æé»è®¤æé å½æ°



### 9.3.6 å®¹å¨æä½å¯è½ä½¿è¿­ä»£å¨å¤±æ

> 9.31 ç¬¬316é¡µä¸­å é¤å¶æ°å¼åç´ å¹¶å¤å¶å¥æ°å¼åç´ çç¨åºä¸è½ç¨äº`list`æ`forward_list`ãä¸ºä»ä¹ï¼ä¿®æ¹ç¨åºï¼ä½¿ä¹ä¹è½ç¨äºè¿äºç±»åã

```cpp
iter+=2;//å¤åèµå¼è¯­å¥ä¸è½ç¨äºè¿ä¸¤ä¸ª
++iter;++iter;//æ¹ä¸ºè¿ä¸ª
```

`forward_list`è¿éè¦åå©å¢å ä¸ä¸ª`prev`æ¥å®æå¢å 



> 9.32 å¨ç¬¬316é¡µçç¨åºä¸­ï¼åä¸é¢è¯­å¥è¿æ ·è°ç¨`insert`æ¯å¦åæ³ï¼å¦æä¸åæ³ï¼ä¸ºä»ä¹ï¼

```cpp
iter = vi.insert(iter, *iter++);
```

ä¸åæ³ï¼åæ°æ±èé¡ºåºä¸ç¡®å®



> 9.33 å¨æ¬èæåä¸ä¸ªä¾å­ä¸­ï¼å¦æä¸å°`insert`çç»æèµäº`begin`ï¼å°ä¼åçä»ä¹ï¼ç¼åç¨åºï¼å»ææ­¤èµå¼è¯­å¥ï¼éªè¯ä½ çç­æ¡ã

è¿­ä»£å¨å¤±æ



> 9.34 åå®`vi`æ¯ä¸ä¸ªä¿å­`int`çå®¹å¨ï¼å¶ä¸­æå¶æ°å¼ä¹æå¥æ°å¼ï¼åæä¸é¢å¾ªç¯çè¡ä¸ºï¼ç¶åç¼åç¨åºéªè¯ä½ çåææ¯å¦æ­£ç¡®ã

```cpp
iter = vi.begin();
while (iter != vi.end())
	if (*iter % 2)
		iter = vi.insert(iter, *iter);
	++iter;
```

ä¸æ­£ç¡®ï¼æ­»å¾ªç¯äº

```cpp
iter = vi.begin();
while (iter != vi.end())
	if (*iter % 2){
			iter = vi.insert(iter, *iter);
			++iter;++iter:
	}else{
		++iter;
	}
```



## 9.4 vector å¯¹è±¡æ¯å¦ä½å¿«éå¢é¿ç

> 9.35 è§£éä¸ä¸ª`vector`ç`capacity`å`size`æä½åºå«ã

`capacity`æ¯å¨ä¸åéæ°çåå­ç©ºé´çåæä¸æå¤å¯ä»¥ä¿å­å¤å°åç´ ï¼`size`æ¯æå·²ç»ä¿å­çåç´ çæ°ç®



> 9.36 ä¸ä¸ªå®¹å¨ç`capacity`å¯è½å°äºå®ç`size`åï¼

ä¸å¯è½



> 9.37 ä¸ºä»ä¹`list`æ`array`æ²¡æ`capacity`æåå½æ°ï¼

`list`é¾è¡¨æ²¡ææ°ééå¶

`array`å¤§å°åºå®äº





> 9.38 ç¼åç¨åºï¼æ¢ç©¶å¨ä½ çæ åå®ç°ä¸­ï¼`vector`æ¯å¦ä½å¢é¿çã

```cpp
#include <iostream>
#include <string>
#include <vector>

using namespace std;

int main()
{
	vector<int> v;

	for (int i = 0; i < 100; i++)
	{
		cout << "capacity: " << v.capacity() << "  size: " << v.size() << endl;
		v.push_back(i);
	}
	return 0;
}
```

`clang version 13.0.0`

```
capacity: 0  size: 0
capacity: 1  size: 1
capacity: 2  size: 2
capacity: 4  size: 3
capacity: 4  size: 4
capacity: 8  size: 5
capacity: 8  size: 6
capacity: 8  size: 7
capacity: 8  size: 8
capacity: 16  size: 9
capacity: 16  size: 10
capacity: 16  size: 11
capacity: 16  size: 12
capacity: 16  size: 13
capacity: 16  size: 14
capacity: 16  size: 15
capacity: 16  size: 16
capacity: 32  size: 17
capacity: 32  size: 18
capacity: 32  size: 19
capacity: 32  size: 20
capacity: 32  size: 21
capacity: 32  size: 22
capacity: 32  size: 23
capacity: 32  size: 24
capacity: 32  size: 25
capacity: 32  size: 26
capacity: 32  size: 27
capacity: 32  size: 28
capacity: 32  size: 29
capacity: 32  size: 30
capacity: 32  size: 31
capacity: 32  size: 32
capacity: 64  size: 33
capacity: 64  size: 34
capacity: 64  size: 35
capacity: 64  size: 36
capacity: 64  size: 37
capacity: 64  size: 38
capacity: 64  size: 39
capacity: 64  size: 40
capacity: 64  size: 41
capacity: 64  size: 42
capacity: 64  size: 43
capacity: 64  size: 44
capacity: 64  size: 45
capacity: 64  size: 46
capacity: 64  size: 47
capacity: 64  size: 48
capacity: 64  size: 49
capacity: 64  size: 50
capacity: 64  size: 51
capacity: 64  size: 52
capacity: 64  size: 53
capacity: 64  size: 54
capacity: 64  size: 55
capacity: 64  size: 56
capacity: 64  size: 57
capacity: 64  size: 58
capacity: 64  size: 59
capacity: 64  size: 60
capacity: 64  size: 61
capacity: 64  size: 62
capacity: 64  size: 63
capacity: 64  size: 64
capacity: 128  size: 65
capacity: 128  size: 66
capacity: 128  size: 67
capacity: 128  size: 68
capacity: 128  size: 69
capacity: 128  size: 70
capacity: 128  size: 71
capacity: 128  size: 72
capacity: 128  size: 73
capacity: 128  size: 74
capacity: 128  size: 75
capacity: 128  size: 76
capacity: 128  size: 77
capacity: 128  size: 78
capacity: 128  size: 79
capacity: 128  size: 80
capacity: 128  size: 81
capacity: 128  size: 82
capacity: 128  size: 83
capacity: 128  size: 84
capacity: 128  size: 85
capacity: 128  size: 86
capacity: 128  size: 87
capacity: 128  size: 88
capacity: 128  size: 89
capacity: 128  size: 90
capacity: 128  size: 91
capacity: 128  size: 92
capacity: 128  size: 93
capacity: 128  size: 94
capacity: 128  size: 95
capacity: 128  size: 96
capacity: 128  size: 97
capacity: 128  size: 98
capacity: 128  size: 99

```



> 9.39 è§£éä¸é¢ç¨åºçæ®µåäºä»ä¹ï¼

```cpp
vector<string> svec;
svec.reserve(1024);//é¢ååé1024ä¸ªåç´ ç©ºé´
string word;
while (cin >> word)
	svec.push_back(word);
svec.resize(svec.size() + svec.size() / 2);//åç´ æ°éåä¸ºåæ¥1.5å
//å¦æåç´ æ°éå 1.5 ååè¶è¿1024åéæ°åéç©ºé´
```



> 9.40 å¦æä¸ä¸é¢çç¨åºè¯»å¥äº256ä¸ªè¯ï¼å¨`resize`ä¹åå®¹å¨ç`capacity`å¯è½æ¯å¤å°ï¼å¦æè¯»å¥äº512ä¸ªã1000ä¸ªãæ1048ä¸ªå¢ï¼

* 256ï¼`capacity=1024`
* 512ï¼`capacity=1024`
* 1000,1048ï¼å¦æä¸¤åå¢é¿ï¼é£ä¹`capacity=2048`



## 9.5 é¢å¤ç string æä½

### 9.5.1 æé  string çå¶ä»æ¹æ³

> 9.41 ç¼åç¨åºï¼ä»ä¸ä¸ª`vector<char>`åå§åä¸ä¸ª`string`ã

```cpp
vector<char>vec={'a','b','c'};
string s(vec.begin(),vec.end());
```



> 9.42 åå®ä½ å¸ææ¯æ¬¡è¯»åä¸ä¸ªå­ç¬¦å­å¥ä¸ä¸ª`string`ä¸­ï¼èä¸ç¥éæå°éè¦è¯»å100ä¸ªå­ç¬¦ï¼åºè¯¥å¦ä½æé«ç¨åºçæ§è½ï¼

ä½¿ç¨ `reserve(100)` å½æ°é¢ååé100ä¸ªåç´ çç©ºé´ã



### 9.5.2 æ¹å string çå¶ä»æ¹æ³

> 9.43 ç¼åä¸ä¸ªå½æ°ï¼æ¥åä¸ä¸ª`string`åæ°æ¯`s`ã`oldVal` å`newVal`ãä½¿ç¨è¿­ä»£å¨å`insert`å`erase`å½æ°å°`s`ä¸­ææ`oldVal`æ¿æ¢ä¸º`newVal`ãæµè¯ä½ çç¨åºï¼ç¨å®æ¿æ¢éç¨çç®åå½¢å¼ï¼å¦ï¼å°"tho"æ¿æ¢ä¸º"though",å°"thru"æ¿æ¢ä¸º"through"ã

```cpp
#include <iostream>
#include <string>
#include <vector>

using namespace std;

void fun(string &s,const string &oldVal,const string &newVal){
	for(auto it = s.begin(); it != s.end();){
		if(oldVal == s.substr(it-s.begin(),oldVal.size())){
			it = s.erase(it,it+oldVal.size());
			it = s.insert(it,newVal.begin(),newVal.end());
			it += newVal.size();
		}else{
			++it;
		}
	}	
}

int main(){
	string s{"123456789"};
	fun(s,"123","567");
	fun(s,"abc","def");
	cout << s << endl;
	return 0;
}
```



> 9.44 éåä¸ä¸é¢çå½æ°ï¼è¿æ¬¡ä½¿ç¨ä¸ä¸ªä¸æ å`replace`ã

```cpp
#include <iostream>
#include <string>
#include <vector>

using namespace std;

void fun(string &s,const string &oldVal,const string &newVal){
	for(size_t pos = 0; pos+oldVal.size() <= s.size(); ++pos){
		if(s.substr(pos,oldVal.size())==oldVal){
			s.replace(pos,oldVal.size(),newVal);
			pos += newVal.size();
		}else{
			++pos;
		}
	}
}

int main(){
	string s{"123456789"};
	fun(s,"123","567");
	fun(s,"abc","def");
	cout << s << endl;
	return 0;
}
```



> 9.45 ç¼åä¸ä¸ªå½æ°ï¼æ¥åä¸ä¸ªè¡¨ç¤ºåå­ç`string`åæ°åä¸¤ä¸ªåå«è¡¨ç¤ºåç¼ï¼å¦"Mr."æ"Mrs."ï¼ååç¼ï¼å¦"Jr."æ"III"ï¼çå­ç¬¦ä¸²ãä½¿ç¨è¿­ä»£å¨å`insert`å`append`å½æ°å°åç¼ååç¼æ·»å å°ç»å®çåå­ä¸­ï¼å°çæçæ°`string`è¿åã

```cpp
#include <iostream>
#include <string>
#include <vector>

using namespace std;

string fun(string& name,const string& pre,const string& suf){
	name.insert(name.begin(),pre.begin(),pre.end());
	name.append(suf);
	return name;
}

int main(){
	string name("Fishingrod");
	cout << fun(name,"Mr.",".zsl");
	return 0;
}
```



> 9.46 éåä¸ä¸é¢çå½æ°ï¼è¿æ¬¡ä½¿ç¨ä½ç½®åé¿åº¦æ¥ç®¡ç`string`ï¼å¹¶åªä½¿ç¨`insert`ã

```cpp
#include <iostream>
#include <string>
#include <vector>

using namespace std;

string fun(string& name,const string& pre,const string& suf){
	name.insert(0,pre);
	name.insert(name.size(),suf);
	return name;
}

int main(){
	string name("Fishingrod");
	cout << fun(name,"Mr.",".zsl");
	return 0;
}
```



### 9.5.3 string æç´¢æä½

> 9.47 ç¼åç¨åºï¼é¦åæ¥æ¾`string`"ab2c3d7R4E6"ä¸­æ¯ä¸ªæ°å­å­ç¬¦ï¼ç¶åæ¥æ¾å¶ä¸­æ¯ä¸ªå­æ¯å­ç¬¦ãç¼åä¸¤ä¸ªçæ¬çç¨åºï¼ç¬¬ä¸ä¸ªè¦ä½¿ç¨`find_first_of`ï¼ç¬¬äºä¸ªè¦ä½¿ç¨`find_first_not_of`ã

```cpp
#include <iostream>
#include <string>
#include <vector>

using namespace std;

int main(){
	string s("ab2c3d7R4E6");
	string num("0123456789");
	for(int i=0;s[i];){
		auto pos = s.substr(i).find_first_not_of(num);
		if(pos!=string::npos){
			cout << s[pos+i];
			i = pos+1+i;
		}else{
			++i;
		}
	}
	cout << endl;
	
	for(int i=0;s[i];){
		auto pos = s.substr(i).find_first_of(num);
		if(pos==string::npos){
			cout << s[i++];
		}else{
			cout <<s.substr(i,pos);
			i = pos+i+1; 
		}
	}
	return 0;
}
```



> 9.48 åå®`name`å`numbers`çå®ä¹å¦325é¡µæç¤ºï¼`numbers.find(name)`è¿åä»ä¹ï¼

`string::npos`



> 9.49 å¦æä¸ä¸ªå­æ¯å»¶ä¼¸å°ä¸­çº¿ä¹ä¸ï¼å¦dæfï¼åç§°å¶æä¸åºå¤´é¨åï¼`ascender`ï¼ãå¦æä¸ä¸ªå­æ¯å»¶ä¼¸å°ä¸­çº¿ä¹ä¸ï¼å¦pægï¼åç§°å¶æä¸åºå¤´é¨åï¼`descender`ï¼ãç¼åç¨åºï¼è¯»å¥ä¸ä¸ªåè¯æä»¶ï¼è¾åºæé¿çæ¢ä¸åå«ä¸åºå¤´é¨åï¼ä¹ä¸åå«ä¸åºå¤´é¨åçåè¯ã

```cpp
#include <fstream>
#include <iostream>
#include <string>

std::string find(const std::string& s,std::string& longest){
	std::string str("aceimnorsuvwxz");
	if(s.find_first_not_of(str)==std::string::npos){
		return (s.size()>longest.size())?s:longest;
	}
	return longest;
}
int main(int argc,char **argv){
	std::ifstream ifs(argv[1]);
	if(ifs){
		std::string longest,s;
		while(ifs>>s){
			longest = find(s,longest);
		}
		std::cout << longest << std::endl;
	}
	return 0;
}
```

### 9.5.5 æ°å¼è½¬æ¢

> 9.50 ç¼åç¨åºå¤çä¸ä¸ª`vector<string>`ï¼å¶åç´ é½è¡¨ç¤ºæ´åå¼ãè®¡ç®`vector`ä¸­ææåç´ ä¹åãä¿®æ¹ç¨åºï¼ä½¿ä¹è®¡ç®è¡¨ç¤ºæµ®ç¹å¼ç`string`ä¹åã

```cpp
#include <fstream>
#include <iostream>
#include <string>
#include <vector>

using std::vector;
using std::string;
using std::cout;
using std::cin;

int main(){
	vector<string>v;
	for(string s;cin>>s;v.push_back(s));
	int sum=0;
	for(auto i:v)sum+=std::stoi(i);
	cout << sum;
	return 0;
}
```



```cpp
#include <fstream>
#include <iostream>
#include <string>
#include <vector>

using std::vector;
using std::string;
using std::cout;
using std::cin;

int main(){
	vector<string>v;
	for(string s;cin>>s;v.push_back(s));
	double sum=0;
	for(auto i:v)sum+=std::stod(i);
	cout << sum;
	return 0;
}
```



> 9.51 è®¾è®¡ä¸ä¸ªç±»ï¼å®æä¸ä¸ª`unsigned`æåï¼åå«è¡¨ç¤ºå¹´ãæåæ¥ãä¸ºå¶ç¼åæé å½æ°ï¼æ¥åä¸ä¸ªè¡¨ç¤ºæ¥æç`string`åæ°ãä½ çæé å½æ°åºè¯¥è½å¤çä¸åçæ°æ®æ ¼å¼ï¼å¦January 1,1900ã1/1/1990ãJan 1 1900 ç­ã

```cpp
#include <iostream>
#include <string>

class Time{
private:
	unsigned year;
	unsigned month;
	unsigned day;

public:
	Time() = default;
	Time(std::string s){
		if(s.find_first_of("/")!=std::string::npos){//1/1/1990
			day = stoi(s.substr(0,s.find_first_of("/")));
			month = stoi(s.substr(s.find_first_of("/")+1,
				(s.find_last_of('/')-s.find_first_of('/'))));
			year = stoi(s.substr(s.find_last_of("/")+1,4));
		}else{
			if( s.find("Jan") < s.size() )  month = 1;
			if( s.find("Feb") < s.size() )  month = 2;
			if( s.find("Mar") < s.size() )  month = 3;
			if( s.find("Apr") < s.size() )  month = 4;
			if( s.find("May") < s.size() )  month = 5;
			if( s.find("Jun") < s.size() )  month = 6;
			if( s.find("Jul") < s.size() )  month = 7;
			if( s.find("Aug") < s.size() )  month = 8;
			if( s.find("Sep") < s.size() )  month = 9;
			if( s.find("Oct") < s.size() )  month =10;
			if( s.find("Nov") < s.size() )  month =11;
			if( s.find("Dec") < s.size() )  month =12;
			
			if(s.find_first_of(" ")!=std::string::npos
			&&s.find_first_of(" ")>3){//January 1,1900
				s[s.find(",")]=' ';
			}

			day = stoi(s.substr(s.find_first_of(" ")+1,
				(s.find_last_of(' ')-s.find_first_of(' '))));
			year = stoi(s.substr(s.find_last_of(" ")+1,4));
		}
	} 

public:
	void print(){
		std::cout << year << " " 
				<< month << " " 
				<< day << std::endl;
	}
};

int main(){
	Time t1("January  1,1900");
	Time t2("Jan 1 1900");
	Time t3("1/1/1900");
	
	t1.print();
	t2.print();
	t3.print();

	return 0;
}
```



## 9.6 å®¹å¨ééå¨

> 9.52 ä½¿ç¨`stack`å¤çæ¬å·åçè¡¨è¾¾å¼ãå½ä½ çå°ä¸ä¸ªå·¦æ¬å·ï¼å°å¶è®°å½ä¸æ¥ãå½ä½ å¨ä¸ä¸ªå·¦æ¬å·ä¹åçå°ä¸ä¸ªå³æ¬å·ï¼ä»`stack`ä¸­`pop`å¯¹è±¡ï¼ç´è³éå°å·¦æ¬å·ï¼å°å·¦æ¬å·ä¹ä¸èµ·å¼¹åºæ ãç¶åå°ä¸ä¸ªå¼ï¼æ¬å·åçè¿ç®ç»æï¼`push`å°æ ä¸­ï¼è¡¨ç¤ºä¸ä¸ªæ¬å·åçï¼å­ï¼è¡¨è¾¾å¼å·²ç»å¤çå®æ¯ï¼è¢«å¶è¿ç®ç»æææ¿ä»£ã

emmä¸å®æ´ççæ ç®è¡¨è¾¾å¼ï¼ä¸é¢è¿ä¸ªç´æ¥æ¯è¡¨è¾¾å¼æ±å¼ç

```cpp
#include<bits/stdc++.h>
using namespace std;

stack<int> num;
stack<char> op;

void eval(){
    auto b=num.top();num.pop();
    auto a=num.top();num.pop();
    auto c=op.top();op.pop();
    int x=0;
    if(c=='+')x=a+b;
    else if(c=='-')x=a-b;
    else if(c=='*')x=a*b;
    else x=a/b;
    num.push(x);
}
int main()
{
    unordered_map<char,int>pr{{'+',1},{'-',1},{'*',2},{'/',2},{'(',0}};//è¿ç®ç¬¦ä¼åçº§
    string s;
    cin>>s;
    for(int i=0;i<s.size();i++)
    {
        auto c=s[i];
        if(isdigit(c))//è½¬æ°å­
        {
            int x=0,j=i;
            while(j<s.size()&&isdigit(s[j]))
                x=x*10+s[j++]-'0';
            i=j-1;
            num.push(x);
        }
        else if(c=='(')op.push(c);
        else if(c==')')//æ¬å·ç´æ¥ç®
        {
            while(op.top()!='(')eval();
            op.pop();
        }
        else
        {
            while(op.size()&&pr[op.top()]>=pr[c])eval();//å¦æå½åä¼åçº§æ¯åé¢çä½å°±ç®
            op.push(c);//æä½ç¬¦å¥æ 
        }
    }
    while(op.size())eval();//å©ä½è®¡ç®
    printf("%d",num.top());
    return 0;
}

```

