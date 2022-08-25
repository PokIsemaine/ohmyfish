# Observer

## 动机

- 在软件构建过程中，我们需要为某些对象建立一种“通知依赖关系” ——一个对象（目标对象）的状态发生改变，所有的依赖对 象（观察者对象）都将得到通知。如果这样的依赖关系过于紧密，将使软件不能很好地抵御变化。
- 使用面向对象技术，可以将这种依赖关系弱化，并形成一种稳定的依赖关系。从而实现软件体系结构的松耦合。

## 模式定义

> 定义对象间的一种一对多（变化）的依赖关系，以便当一个对象(Subject)的状态发生改变时，所有依赖于它的对象都得到通知并自动更新。 
>
> <p align="right">——《 设计模式》 GoF</p>



## SHOW ME THE CODE

以文件分割器为例

`MainForm1.cpp`界面

```cpp
class MainForm : public Form
{
	TextBox* txtFilePath;
	TextBox* txtFileNumber;
	ProgressBar* progressBar;//进度条

public:
	void Button1_Click(){

		string filePath = txtFilePath->getText();
		int number = atoi(txtFileNumber->getText().c_str());

		FileSplitter splitter(filePath, number, progressBar);

		splitter.split();

	}
};
```

`FileSplitter1.cpp`文件分割器

```cpp
class FileSplitter
{
	string m_filePath;
	int m_fileNumber;
	ProgressBar* m_progressBar;

public:
	FileSplitter(const string& filePath, int fileNumber, ProgressBar* progressBar) :
		m_filePath(filePath), 
		m_fileNumber(fileNumber),
		m_progressBar(progressBar){

	}

	void split(){

		//1.读取大文件

		//2.分批次向小文件中写入
		for (int i = 0; i < m_fileNumber; i++){
			//...
			float progressValue = m_fileNumber;
			progressValue = (i + 1) / progressValue;
			m_progressBar->setValue(progressValue);//更新进度条
		}

	}
};
```

我们希望展示分割的进度，所以我们需要在`split()`当中去更新我们的进度



### 利用设计原则发现问题

上面这种写法看起来没啥问题，当我们采用设计原则去检验的时候发现其违反了依赖倒置原则

依赖倒置（DIP）：编译时依赖，A 依赖 B 指 A 编译时 B 必须存在

* 高层模块（稳定）不应该依赖于低层模块（变化），二者都应该依赖于抽象（稳定）
* 抽象（稳定）不应该依赖于实现细节（变化），实现细节应该依赖于抽象（稳定）



```cpp
class MainForm : public Form
{
	TextBox* txtFilePath;
	TextBox* txtFileNumber;
	ProgressBar* progressBar;//这里其实是具体实现细节了，产生了编译时依赖

public:
	void Button1_Click(){

		string filePath = txtFilePath->getText();
		int number = atoi(txtFileNumber->getText().c_str());

		FileSplitter splitter(filePath, number, progressBar);

		splitter.split();

	}
};
```

为什么`progressBar`是实现细节？

其实可以想想我们身边的进度条设计，他们可能多样的，我们没法确保以后也是用这种方式或`UI`去进展现，因此这个实际上是一个实现细节。

`MainForm`现在依赖了`progressBar`的实现细节，稳定依赖变化



### 优化尝试：找基类

依赖倒置原则一个直观的理解是：不要依赖 A，而要依赖 A 的抽象基类。

那么，我们可以通过找基类的方式解决这个问题吗？

如果我们弄一个`ControlBase`作为`progressBar`的抽象基类，会发现它并没有`setValue`之类的方法，走到死胡同了

`progressBar`实际上扮演的是一个通知的角色，我们可以用一个抽象的方式来表达通知，而不是一个具体控件

### 优化：抽象通知机制

我们将一个具体的通知部件转为一个抽象的通知机制

`FileSplitter2.cpp`

```cpp
class IProgress{
public:
	virtual void DoProgress(float value)=0;
	virtual ~IProgress(){}
};


class FileSplitter
{
	string m_filePath;
	int m_fileNumber;
	//ProgressBar* progressBar; 具体的通知部件
	List<IProgress*>  m_iprogressList; // 抽象通知机制，支持多个观察者
	
public:
	FileSplitter(const string& filePath, int fileNumber) :
		m_filePath(filePath), 
		m_fileNumber(fileNumber){

	}


	void split(){

		//1.读取大文件

		//2.分批次向小文件中写入
		for (int i = 0; i < m_fileNumber; i++){
			//...

			float progressValue = m_fileNumber;
			progressValue = (i + 1) / progressValue;
			onProgress(progressValue);//发送通知
		}

	}


	void addIProgress(IProgress* iprogress){
		m_iprogressList.push_back(iprogress);
	}

	void removeIProgress(IProgress* iprogress){
		m_iprogressList.remove(iprogress);
	}


protected:
	virtual void onProgress(float value){
		
		List<IProgress*>::iterator itor=m_iprogressList.begin();

		while (itor != m_iprogressList.end() )
			(*itor)->DoProgress(value); //更新进度条
			itor++;
		}
	}
};
```

`MainForm2.cpp`

```cpp
class MainForm : public Form, public IProgress//多继承，一个是主继承类其他都是抽象基类/接口
{
	TextBox* txtFilePath;
	TextBox* txtFileNumber;

	ProgressBar* progressBar;

public:
	void Button1_Click(){

		string filePath = txtFilePath->getText();
		int number = atoi(txtFileNumber->getText().c_str());

		ConsoleNotifier cn;

		FileSplitter splitter(filePath, number);

		splitter.addIProgress(this); 
		splitter.addIProgress(&cn); 

		splitter.split();

		splitter.removeIProgress(this);

	}

	virtual void DoProgress(float value){
		progressBar->setValue(value);
	}
};

class ConsoleNotifier : public IProgress {
public:
	virtual void DoProgress(float value){
		cout << ".";
	}
};
```

* 多继承一般不推荐用，这里的话比较特殊`MainForm`是一个主要类继承其他都是接口/抽象基类继承

现在我们可以看到`FileSplitter`不再依赖`MainForm`了



回顾一下我们的路线：朴素实现-->利用设计原则发现问题-->考虑通过找基类来优化-->考虑使用`Observer`设计模式

## 结构设计

![image-20220215193551596](https://s2.loli.net/2022/02/15/smkXwvHFLcgxOy2.png)



## 要点总结

- 使用面向对象的抽象，Observer 式使得我们可以独立地改变目标与观察者，从而使二者之间的依赖关系达致松耦合。
- 目标发送通知时，无需指定观察者，通知（可以携带通知信息作为参数）会自动传播。
- 观察者自己决定是否需要订阅通知，目标对象对此一无所知。
- Observer 模式是基于事件的 UI 框架中非常常用的设计模式，也是 MVC 模式的一个重要组成部分。