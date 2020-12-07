---
title: 面向对象实验——solitaire纸牌游戏
date: 2020-04-22 19:52:34
categories: 
- 课程设计
tags:
- java
---
单人纸牌游戏，牌桌上有7个堆共28张牌，第一堆1张牌，第二堆2张，。。。第7堆7张，每一堆的第一张牌朝上，其他朝下。牌桌上还有4个suitpiles，一个deck card堆和一个discard card堆，布局如下（参考windows的纸牌游戏）
<!-- more -->
## 项目地址
https://github.com/ccclll777/Windows_Solitaire_game
**如果有帮助可以点个star**
## 实验内容
使用java/C++语言，利用面向对象技术，模拟windows纸牌小游戏。
单人纸牌游戏，牌桌上有7个堆共28张牌，第一堆1张牌，第二堆2张，。。。第7堆7张，每一堆的第一张牌朝上，其他朝下。牌桌上还有4个suitpiles，一个deck card堆和一个discard card堆，布局如下（参考windows的纸牌游戏）。![在这里插入图片描述](https://img-blog.csdnimg.cn/2020042708515240.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
## 总体功能设计
**一．需求分析**
单人纸牌游戏，牌桌上有7个堆共28张牌，第一堆1张牌，第二堆2张……第7堆7张，每一堆的第一张牌朝上，其他朝下。牌桌上还有4个suitstack，一个deck card堆和一个discard card堆。初始时四个suitstack和一个deck card堆为空，discard card堆中有24张牌且全部背面朝下，在明确规则之后，需要设计图形界面，增加与用户的交互性。为了用户提供更好的游戏体验，可以为用户提供不同难度的纸牌游戏，重新定义发牌规则。

**二．目标分析**
   首先明确游戏的规则，游戏共有十三个牌堆，十三个牌堆被分为四个区域：桌面堆(TableStack),花色堆(SuitStack)，发牌堆（DeckStack）,和丢弃堆（DisStack）

桌面堆一共有7个牌堆，纸牌的数量是从1到7共28张，每一个桌面堆初始状态是，只有最底端的牌为正面，其他牌为反面。
花色堆初始状态为空。
丢弃堆初始状态也为空。
其他剩下的24张牌放入发牌堆且牌面朝下。

**游戏规则：**
（1）每次点击发牌堆，会将一张牌加入丢弃堆，表示发了一张牌。
（2）在桌面堆的玩牌规则：如果要将牌拖向桌面堆，需要满足的是，拖动牌堆的最顶端，与放入的牌堆的最底端的颜色不同且大小小1，满足这个条件的牌堆，就可以合在一起。
（3）从丢弃堆向桌面堆拖动牌也需要满足同样的要求，拖动牌堆的最顶端，与放入的牌堆的最底端的颜色不同且大小小1。
（4）花色堆有四个，每一个放一种花色的牌，然后从一开始存放，每次只能拖动一个到花色堆。需要满足的条件是，花色堆顶端的卡牌的花色和需要拖进去卡牌的花色相同，且大小小1.如果满足这个条件，就可以将牌放入花色堆。

**游戏功能：**
（1）实现几个牌堆的交互，如果符合规则的话可以互相拖动。
（2）如果发牌堆变为空但是丢弃堆没空的话，说明牌没有发完，这时候把丢弃堆的牌在放入发牌堆，然后重新发一遍。直到全部发完为止。
（3）如果用户想重新开始一局新的游戏，可以点击重新启动的按钮，然后重新发牌，开始一局新的游戏。
（4）为用户提供两个等级的难度。low和high。
low难度表示，如果一个桌面堆是空的，你可以把其他任意牌拖到桌面堆。
high难度表示，如果一个桌面堆是空的，他这里只能放从K 开始的牌堆。
low难度可以放松对用户的约束，让用户可以更容易的通关。
（5）如果你通关，可以为用户显示通关成功之类的东西。

## 设计思路

 - **有关牌堆类的设计**

**(1）枚举类Suit和Rank**
我们需要纸牌的四个花色和十三个大小，所以初始化了两个枚举类Suit和Rank，分别存放花色和大小，以便卡牌的初始化。

**(2）Card类**
我们需要卡牌类Card，然后初始化每一张卡牌，Card类实现了Card接口，Card类中有花色，大小，是否正面朝上的属性，以及各个属性对应的get和set方法。还提供了一个静态的方法，在项目编译时会进行初始化，创建了一个数组，里边存了每个花色的每个纸牌的编号，在项目运行过程中，可以通过卡牌编号寻找是那张纸牌，也可以通过卡牌的花色和大小找出卡牌编号，方便我们使用。（静态类和final类型的变量，在编译时就已经创建，并且初始化）

**（3）CardStack**
由于牌堆时从一头进行出入的，并且是先进后出的，所以我们使用了 ArrayList来模拟了堆栈进行操作。首先创建CardStack类实现CardStack_Interface接口，它是十三个牌堆的父类，它创建了十三个牌堆的属性，实现了牌堆的基本方法。它的基本属性有存放卡牌的ArrayList，实现的方法有，获取ArrayList的大小，获取最顶部卡牌，往牌堆顶添加一张卡牌，获取指定位置的牌，push一组卡牌，删除一组卡牌，删除顶端卡牌，清空牌堆等等方法。

**(4)DescStack发牌堆**
继承自CardStack，由于添加时需要设置纸牌的背面朝上，所以需要重写父类的push方法。

**(5)DiscardStack丢弃堆**
继承自CardStack，由于向 DisCardStack添加卡牌时都是朝上的，所以重写了父类的push操作。

**(6)SuitStack花色堆**
花色堆继承自CardStack，是收集已经有序的牌的地方，由于进入花色堆的牌是正面朝上的，所以重写了父类的push操作，让进入牌堆的牌正面朝上。
**(7)TableStack桌面堆**
桌面堆是玩牌的主要区域，需要根据游戏的规则堆牌堆进行拖动，达到排序的效果，最终让牌全部放到花色堆。桌面堆继承自CardStack，并且添加了两个其他的方法：
第一可以根据花色和大小，确定这张牌在这个牌堆的位置，在牌堆拖动时，可以用来确定这是那一张牌。
第二可以获得这个牌堆的子牌堆，根据第一个方法获取到这个牌子啊牌堆的位置，将这个牌堆内大小比这个牌都小的牌返回，实现牌堆间的拖动。

**

 - 有关游戏逻辑的设计（各个牌堆内的配合）

**
类GameModel实现了有关游戏规则，它是一个final类，在程序编译时将被创建并且初始化，并且该类不能被继承。

在 GameModel类中，对于游戏的十三个牌堆进行了初始化，将十三个牌堆加入泛型为CardStack的ArrayList的中进行管理。

牌堆初始化时，将28张牌分别放入桌面堆中，然后让他们最顶端的卡牌朝上，然后将剩余的24张牌放入发牌堆。

由于GameModel是一个final类，初始化后不能改变，所以在GameModel类中提供一个方法，他是一个静态方法，可以通过类名加函数名访问，返回值是GameModel类的一个实例，这样在类外，就可以通过获取的这个实例来使用这个类中的其他方法了。

GameModel中提供了有关游戏牌堆拖动是否合法的判断，如果拖动合法，则进行牌堆拖动的逻辑。

GameModel中实现了，发牌的逻辑，每次将发牌堆DescStack中最顶端的牌加入到DisCardStack中。

GameModel实现了游戏的重新发牌的调用，实现了如果发牌堆变为空但是丢弃堆没空的话，说明牌没有发完，这时候把丢弃堆的牌在放入发牌堆，然后重新发一遍。直到全部发完为止。

GameModel实现了将牌堆中的牌，通过格式化变成字符串，在拖动时可以放到剪切板内，然后传到目的牌堆上，可以对传递的牌堆进行一系列的处理，看拖动是否合法，是否可以完成相应的拖动。

GameModel实现了对于每一个牌堆的监听功能，如果牌堆发生改变，监听器会调用所有牌堆界面类的绘制操作，对于桌面进行重新绘制。

GameModel提供了设置游戏难度的方法，可以将游戏设置成不同的难度。

**

 - 有关游戏界面类的设计

**
**（1）定义一个CardImage类**
里面定义两个方法，一个根据对应的card获取牌的图片，另一个获取牌反面的图片。
**（2）游戏的界面使用javafx进行绘制**
将每一个牌堆的界面都设计为一个界面类，共有SuitPileView(花色堆界面类)，DescPileView（发牌堆界面类），DiscardPileView（丢弃堆界面类），TablePileView（桌面堆界面类）。

**DescPileView（发牌堆界面类）**
发牌堆界面类继承字Hbox,然后每次初始化时，初始化一个按钮，按钮的外观是纸牌的背面，并且给按钮设置点击事件，每点击一次发牌堆的纸牌，就会将发牌堆顶端的牌放入丢弃堆，实现发牌功能。

**DiscardPileView（丢弃堆界面类）**
里面卡牌都正面朝上放在一起，每次拖走一张会显示下面的一张，如果没有牌时牌堆为空，每次点击发牌，都会增加一张。

**SuitPileView(花色堆界面类)**
每次有一个牌放入花色堆，都会在界面上绘制出来。

**TablePileView（桌面堆界面类）**
在初始化时，根据每张牌的正反进行初始化，如果是正，则屏幕上显示正面，如果是反面，则屏幕上显示是卡牌的背面呢，可以支持牌堆互相之间的拖动，如果符合规则会保留在上面。

**（3）下面说明牌堆拖动的逻辑**
以上所有牌堆的拖动逻辑大致相同，只是放置的规则不同，调用的是不同的函数判断是否可以放置，可以为每一张image进行，设置拖动的事件。拖动事件有以下几种触发方式：
setOnDragOver：当你拖动到目标上方时会执行。
setOnDragEntered：当你进入目标控件时会执行
setOnDragExited：当你移出一个目标控件时会执行
setOnDragDropped：当你移动到目标控件，并且松开鼠标时会执行
setOnDragDetected：当你在某个控件上移动时，会监测到拖动操作，执行这个函数。
setOnDragDetected中的判断逻辑，当鼠标拖动这个项目时，初始化一个拖放操作，然后创建一个剪切板，将你拖动的所有牌，获取它对应的名称（初始化card类时，创建了一个fnial的静态数组，为每个牌初始化了一个名称），然后将他们拼接在一起，放到剪切板中，以便其他牌堆获取拖动内容。
setOnDragOver中的判断逻辑：如果你拖动一个牌堆到另一个牌堆上方时，会不断执行这个函数，他会根据规则判断此次拖动是否合法（根据桌面堆和花色堆不同的规则），如果合法则返回true，可以进行拖动操作。
setOnDragExited中逻辑，当一堆牌移动出目标控件时，会执行这个调用，设置卡牌的效果。
setOnDragEntered，当牌堆进入另一个牌堆时，会执行这个调用，为卡牌设置一些效果。
setOnDragDropped：当你移动到目标控件，并且松开鼠标时会执行,根据之前移动是否合法的判断，来看是否可以放置在这个牌堆上，它在两个牌堆之间传递了剪切板上的内容，首先将剪切板上的内容，转化为一个牌堆，如果可以移动，就在目的牌堆中添加这个牌堆，在原来牌堆中删除这个牌堆，否则不进行任何操作。

**（4）定义了一个接口GameModelListener**
里面定义了一个方法gameStateChanged() 表示界面做了更新。
我们让SuitPileView(花色堆界面类)，DescPileView（发牌堆界面类），DiscardPileView（丢弃堆界面类），TablePileView（桌面堆界面类）都实现这个接口。然后里面调用了更新各自界面的方法。
在GameModel中初始化了一个arraylist  存放的GameModelListener为泛型的变量，然后将每一个初始化的牌堆都加入这个arraylist中，当有界面更新时，通过遍历这个arraylist就可以调用所有界面内的gameStateChanged（）的函数，实现界面的同步更新。
    
 （5）最终在主界面Solitaire_GUI中对于以上类进行初始化以及统一的绘制，主界面使用GridPand进行设计，GridPand是一种网格的布局方式，卡牌正好和网格相对应，每一个牌堆可以放在不同的网格之中，所以之后的牌堆都被初始化在了一个个的网格中，会比较整齐。
在Solitaire_GUI中对于每个牌堆进行了初始化，共初始化了13个牌堆的View界面，还有调整游戏难度，以及重新发牌的按钮。

## 代码实现
**具体的代码看github**
https://github.com/ccclll777/Windows_Solitaire_game
**一．有关Rank和Suit枚举类的实现**	
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427090638477.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
Rank为卡牌大小的枚举类，从ACE-KING
Suit为卡牌花色的枚举类。
**二．有关card_Interface接口和card类的实现**![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427090653625.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
Card_Interface定义了五个属性，分别为牌的花色Suit，大小Rank，牌面是否朝上faceUp，以及牌面是否为红色isRed。
定义了卡牌的五个方法，‘
获取花色getSuit()
获取卡牌大小getRank()
牌面是否朝上isFaceUp()
牌面是否为红色isRed()
还有设置牌是否正面朝上setFaceUp（）。
Card类实现了接口中的方法，并且增加了一个final static的数组CARDS，在初始化时会初始化所有的52张牌到一个二维数组中。
并且可以通过花色和大小获取相应的牌 Card get(Rank pRank, Suit pSuit) 
以及根据编号获取相应的牌Card get(String pId)

```java
private  final Rank aRank;//大小
private  final Suit aSuit;//花色
private boolean faceUp = false;//扑克牌是否正面朝上
private boolean isRed ;
//方法为共有的
//52张牌的初始化
  private static final Card[][] CARDS = new Card[Suit.values().length][];
  // 初始化每一张牌的花色和大小，生成一个编号，以便可以通过编号获取相对应的卡牌
static
{
    for( Suit suit : Suit.values() )
    {
        CARDS[suit.ordinal()] = new Card[Rank.values().length];
        for( Rank rank : Rank.values() )
        {
            CARDS[suit.ordinal()][rank.ordinal()] = new Card(rank, suit);
        }
    }
}
//构造方法
public Card(Rank pRank, Suit pSuit)
{
    aRank = pRank;
    aSuit = pSuit;
    if(pSuit == Suit.HEARTS || pSuit == Suit.DIAMONDS)
    {
        isRed = true;
    }
    else
    {
        isRed = false;
    }

}
//获取这张牌的编号
public String getIDString()
{
    return Integer.toString(getSuit().ordinal() * Rank.values().length + getRank().ordinal());
}

//返回这张牌是否朝上
public boolean isFaceUp() {
    return faceUp;
}
//设置这张牌的朝向
public void setFaceUp(boolean faceUp) {
    this.faceUp = faceUp;
}
//返回是否为红色
public boolean isRed() {
    return isRed;
}
//获得花色
public Suit getSuit() {
    return this.aSuit;
}
//根据编号获取对应的牌
public static Card get(String pId)
{ if(pId != null)
    {
        int id = Integer.parseInt(pId);
        return get(Rank.values()[id % Rank.values().length],
                Suit.values()[id / Rank.values().length]);
    }

        return null; }
//根据花色和大小获取对应的牌
public static Card get(Rank pRank, Suit pSuit)
{
    if( pRank != null && pSuit != null)
    {
        return CARDS[pSuit.ordinal()][pRank.ordinal()];
    }

        return null;
}
//获取牌的大小
public Rank getRank() {
    return this.aRank;
    }
```
**三．牌堆接口以及牌堆类（Card_Interface 和CardStack以及它的子类。）**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427090739905.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
（1）CardStack_Interface中定一个一个属性pokers_card，里面需要存这个牌堆的所有卡牌。
然后定义了以下方法：
int size();获取牌堆的发小
ArrayList<Card> getPokers_card();获取牌堆的实例
void init(Card pCard);向牌堆顶端加一张牌（表示桌面堆最下面的一张）
Card peek();获取最顶端的牌
Card peek(int index);//获取指定位置的牌
boolean isEmpty();牌堆是否为空
void push(CardStack pStack, int index);在指定位置添加一组牌
void pop(Card pCard);删除一张卡牌
void pop();删除顶端卡牌
void clear();清空牌堆。
这里的peek（）方法和pop（）方法都实现了重载解析，他有不同的参数列表
（2）CardStack类实现了CardStack_Interface接口。
实现了它的所有方法

```java
public CardStack()
{
    pokers_card = new ArrayList<Card>();
}
//获取list的大小
public int size() {
    return this.pokers_card.size();
}
//获取顶部卡片
public ArrayList<Card> getPokers_card()
{
    return this.pokers_card;
}
public Card peek() {
//如果牌堆有牌则获得顶部卡牌，否则获得一张红桃A
   if(this.pokers_card.size() >0)
   {
       return this.pokers_card.get(this.pokers_card.size() - 1);
   }
   else
   {
       return new Card(Rank.ACE,Suit.HEARTS)   }
  }
   //给牌堆添加一张卡牌，添加到最顶端
public void init(Card pCard)
{
    this.pokers_card.add(pCard);
}
//获取指定位置的卡片
public Card peek(int index)
{

    if(index >= 0 && index < size())
    {
        return this.pokers_card.get(index);
    }
return null;}
//list是否为空
public boolean isEmpty() {

    return pokers_card.size() == 0;

}
//push一组卡牌
public void push(CardStack pStack, int index) {

    for(int i = index; i < pStack.size(); i++) {
        this.pokers_card.add(pStack.peek(i));
    }

}

public void push(CardStack pStack) {
    for(int i = 0; i < pStack.size(); i++) {
        this.pokers_card.add(pStack.peek(i));

    }

}
//删除一组卡牌
public void pop(Card pCard) {
    if(!isEmpty())
    {
        for(int i = 0 ; i <this.pokers_card.size() ; i++)
        {
            if(pCard.getSuit() == this.pokers_card.get(i).getSuit() &&pCard.getRank() == this.pokers_card.get(i).getRank() )
                this.pokers_card.remove(i);
            }
    }

}
//删除顶部卡牌

public void pop() {
    if(!isEmpty())
        this.pokers_card.remove(pokers_card.size()-1);
    }


public void clear() {
    this.pokers_card.clear();
}

```
（3）DescStack继承自CardStack类，它继承了来自CardStack的所有方法，由于发牌堆与普通牌堆没有区别，都是背面朝上的牌堆，所以不需要对牌堆进行重写。
（4）DiscardStack继承自CardStack类，它继承了来自CardStack的所有方法，由于丢弃堆最顶端的牌需要正面朝上，所以它重写了父类的push方法，需要在添加卡牌后将牌堆的第一张牌的isfaceup设置成true。

```java
public void push(CardStack pStack, int index) {
    super.push(pStack, index);
    //添加一组卡牌后   卡牌的顶部应该是朝上的
  if(!this.isEmpty())
      this.peek().setFaceUp(true);
  }
```
（5）SuitStack继承自CardStack类，它继承了来自CardStack的所有方法
,由于花色堆的所有牌的正面都是朝上的，所以重写了父类的push方法，在添加卡牌后，将添加的所有卡牌的正面全部朝上。

```java
public void push(CardStack pStack, int index) {
    super.push(pStack, index);
    if (!this.isEmpty()) {
        this.peek().setFaceUp(true);
    }
}
```
（6）TableStack继承自CardStack类，它继承了来自CardStack的所有方法
由于桌面堆需要获取子牌堆（一组牌可以同时拖动，可以将他们一同获取），所以需要添加一个获取子牌堆的方法。
我们需要获取一张牌是这个牌堆中的第几个，所以添加一个获取这个牌是牌堆中第几个的方法。（其实第二个方法完全为第一个方法服务）

```java
public int  getCardIndex(Rank pRank, Suit pSuit) {

    if(!this.isEmpty())
    {
 for(int i = this.size() -1 ; i >= 0 ; i--)
        {
            if(this.peek(i).getRank() == pRank && this.peek(i).getSuit() == pSuit)
                    return i;
        }
    }

    return -1;
}
public TableStack getSubStack(Card pCard, TableStack stack)
{
    if(pCard != null)
    {
        int index = stack.getCardIndex(pCard.getRank(),pCard.getSuit());
        TableStack temp_stack = new TableStack();
        for(int i = index ; i< stack.size() ; i++)
        {
            temp_stack.init(stack.peek(i));

        }
        return temp_stack;

    }
    return null;
}
```
**四．牌堆的主要拖动的逻辑实现类GameModel**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427090936942.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
GameModel中对十三个牌堆进行了实例化， 然后将它们存入一个泛型为CardStack的ArrayList中。这样已经体现出了面向对象的优越性，可以将子类的对象放入父类变量的泛型之中，便于管理。
   并且GameModel是一个final类，在程序编译时就已经初始化，随机出了桌面堆和发牌堆的所有纸牌，在GameModel实现了对于纸牌拖动的一些逻辑的处理过程。
GamoModel中的属性：

private static final GameModel INSTANCE = new GameModel();在final类中实例化了一个GameModel的变量，然后使用静态方法，将它返回到其他类中，实现通过类名使用GameModel中的函数。

 private  ArrayList<CardStack> card_Stacks 存放十三个牌堆ArrayList.

private int fromIndex;牌堆从哪里移动的
private DeckStack deck_Stack;//0 /／ArrayList中的第1个牌堆，发牌堆
private DisCardStack disCard_Stack;//1ArrayList中的第2个牌堆，丢弃堆
private TableStack[] table_Stacks;//2-8  ArrayList中的第2个到第8个牌堆，桌面堆

private SuitStack[] suit_Stacks;//9-12 ArrayList中的第9个到第12个牌堆，花色堆

private static final String SEPARATOR = ";";//进行字符串分割的符号
private String level = "low";//设置游戏难度值的符号。
private final List<GameModelListener> aListeners = new ArrayList<GameModelListener>();  //存放GameModelListener类的ArrayList，作为所有牌堆的监听器，实现触发后界面的更新。

GameModel中实现的方法：
Void init():初始化十三个牌堆，给桌面堆和玩牌堆随机发牌。将他们每一个牌堆按顺序存入ArrayList中，之后根据序号访问不同的牌堆。
CardStack getStack（）可以根据序号从ArrayList中取出自己需要的牌堆，然后返回。实现了内部数据的保护。
GameModel instance()//在初始化时，初始化了GameModel的实例，通过这个静态方法，将GameModel的实例返回，在类外，可以通过类名+函数名调用这个方法，获取GameModel的实例，使用GameModel的函数，方便用户的使用。
void addListener（）  为每一个牌堆的界面类设置监听器。
void notifyListeners()  调用时，调用每一个牌堆更新界面的方法，实现界面的同步更新。
CardStack getSubStack（） 获得一个桌面堆的子牌堆，在拖动时可以拖动一堆卡牌。
boolean isLegalMove（）  判断牌在牌堆之间的移动是否符合规则。
boolean  canMoveToTableStack （） 判断移动到桌面堆的牌堆是否符合规则。
boolean  canMoveToSuitStack（） 判断移动到花色堆的牌是否符合规则
 void Desc_to_DisCard()  模拟发牌，每次点击发牌按钮，将一张牌从发牌堆加入丢弃堆
boolean moveCard（） 牌堆移动的主要逻辑，如果一组卡牌能在两个牌堆之间移动，那么通过调用这个类，将这一组卡牌从起始牌堆删除，然后添加到目的牌堆
void pop_from（） 将一组卡牌从起始牌堆删除
void reset_all()   当发牌堆和丢弃堆都没有卡牌时，重新发牌，开始一局新的游戏。
 void reset()   当丢弃堆还有卡牌，发牌堆没有卡牌时，将丢弃堆的牌重新放入发牌堆，重新发牌。
CardStack StringToStack  由于需要移动的卡牌在牌堆间的传递，是通过在剪切板中存放字符串实现牌堆间信息的传递的，这里将字符串中表示的卡牌转化成真实的牌堆。
Card getTop（）获取剪切板中字符串表示的卡牌中，最顶端的一张卡牌。
String serialize（）//将牌堆转换为字符串的方法，传入一组牌堆，通过过去每一张牌的唯一名字，获取卡牌的名称，拼接在字符串中。
void setFromIndex（）设置牌堆拖动时的起始牌堆
setLevel(String level)  设置游戏的难度

```java
在这里插入代码片
```

```java
public void init()
 {
     this.table_Stacks = new  TableStack[7];
     for(int i = 0 ; i < table_Stacks.length ; i++)
     {
         this.table_Stacks[i] = new TableStack();
     }
     this.deck_Stack = new DeckStack();
     this.disCard_Stack = new DisCardStack();
     this.suit_Stacks = new SuitStack[4];
     for(int i = 0 ; i< suit_Stacks.length ; i++)
     {
         this.suit_Stacks[i] = new SuitStack();
     }
     this.card_Stacks = new ArrayList<CardStack>();
     this.card_Stacks.add(this.deck_Stack);//0
     this.card_Stacks.add(this.disCard_Stack);//1
     for(int i = 0 ; i < table_Stacks.length ; i ++)
     {
         this.card_Stacks.add(this.table_Stacks[i]);//2-8
     }
     for(int i = 0 ; i < suit_Stacks.length ; i ++)
     {
         this.card_Stacks.add(this.suit_Stacks[i]);//9-12
     }
      Random random = new Random();
     ArrayList<Card> normal_Rank = new ArrayList();
     Card temp_card = null;
     // 使用normal_Rank暂存52张卡牌的信息
     for( Suit suit : Suit.values() )
     {
         for( Rank rank : Rank.values() )
         {
             temp_card = new Card(rank, suit);
             normal_Rank.add(temp_card);
         }
     }
     int i;
     //随机主七个主牌堆的牌
     int index = 0;
     for( i = 0; i < 7; ++i) {
         for(int j = 0; j <= i; ++j) {
             while (true)
             {
                 index = random.nextInt(normal_Rank.size());
                 temp_card = (Card)normal_Rank.get(index);
                 if( !this.table_Stacks[i].isEmpty() && temp_card.isRed() != this.table_Stacks[i].peek().isRed())
                 {
                     this.table_Stacks[i].init(temp_card);
                     normal_Rank.remove(index);
                     break;
                 }
                 if(this.table_Stacks[i].isEmpty() )
                 {
                     this.table_Stacks[i].init(temp_card);
                     normal_Rank.remove(index);
                     break;
                 }
             }
         }
     }
     //将每个牌堆的顶端翻正
     for(i = 0; i < 7; ++i) {
         this.table_Stacks[i].peek().setFaceUp(true);
     }
     //将剩余牌放入发牌堆
     for(i = 0; i < 24; ++i) {
         index = random.nextInt(normal_Rank.size());
         temp_card = (Card)normal_Rank.get(index);
         this.deck_Stack.init(temp_card);
         normal_Rank.remove(index);
     }
 }
 //获取牌堆
 public CardStack getStack(int index) {
     return this.card_Stacks.get(index);
 }
 //获取实例
 public static GameModel instance()
 {
     return INSTANCE;
 }
 //设置卡牌移动的监听器
 public void addListener(GameModelListener pListener)
 {
    if(pListener != null);
     aListeners.add(pListener);
 }
 //在卡牌移动后，更新桌面
 private void notifyListeners()
 {
     for( GameModelListener listener : aListeners )
     {
         listener.gameStateChanged();
     }
 }
 //返回卡牌的子序列，点击桌面堆的七个牌堆的子序列。
 public CardStack getSubStack(Card pCard, int aIndex)
 {
     TableStack stack = (TableStack) GameModel.instance().getStack(aIndex);
     TableStack temp_stack = stack.getSubStack(pCard,stack);

         return temp_stack;

 }
 //判断 移动是否合法
 public boolean isLegalMove(Card pCard,int aIndex )
 {
     if(aIndex >=1 && aIndex<= 8)
     {
             return canMoveToTableStack(pCard,aIndex);
     }
     else if(aIndex>= 9 && aIndex <=12)
     {
             return canMoveToSuitStack(pCard,aIndex);
     }

     return false;
 }
 //是否能移到桌面堆
 public boolean  canMoveToTableStack(Card pCard,int aIndex)
 {
     if(level == "high")
     {
         if(pCard!=null)
         {
             CardStack temp_stack = getStack(aIndex);
             if( temp_stack.isEmpty() )
             {

                 return pCard.getRank() == Rank.KING;
             }
             else
             {
                 return pCard.getRank().ordinal() == temp_stack.peek().getRank().ordinal()-1 &&
                         pCard.isRed() != temp_stack.peek().isRed();
             }

         }
     }
     else if(level == "low")
     {

         if(pCard!=null)
         {
             CardStack temp_stack = getStack(aIndex);
             if( temp_stack.isEmpty() )
             {

                 return true;
             }
             else
             {
                 return pCard.getRank().ordinal() == temp_stack.peek().getRank().ordinal()-1 &&
                         pCard.isRed() != temp_stack.peek().isRed();
             }

         }
     }

     return false;
 }
 //是否能移到花色堆堆
 public boolean  canMoveToSuitStack(Card pCard,int aIndex)
 {
     assert pCard != null ;
     CardStack temp_stack = getStack(aIndex);
     if(temp_stack.isEmpty())
     {
         return  pCard.getRank() == Rank.ACE ;
     }
     else
     {
         return pCard.getRank().ordinal() == temp_stack.peek().getRank().ordinal()+1 &&
                 pCard.getSuit()==temp_stack.peek().getSuit();
     }


 }

 public void Desc_to_DisCard()
 {
     Card temp_card = this.getStack(0).peek();
     this.getStack(1).init(temp_card);
     this.getStack(0).pop();
     notifyListeners();

 }
 //移动牌堆
     public boolean moveCard(CardStack from,int aIndex)
     {
         if
             (aIndex>=2 && aIndex<=8)
         {
             TableStack to = (TableStack)this.getStack(aIndex);
             if (!to.isEmpty())
             {
                 this.getStack(aIndex).push(from);
                 pop_from(from);
                 notifyListeners();
                     return true;


     } else if (from.isEmpty()) {

         return false;
     } else {

                     this.getStack(aIndex).push(from);
                     pop_from(from);
                     notifyListeners();
}
         }
         else  if(aIndex>=9 && aIndex<= 12)
         {
             SuitStack to = (SuitStack) this.getStack(aIndex);
             if (to.isEmpty())
             {//获取他的下个一卡牌
                 this.getStack(aIndex).push(from);
                 pop_from(from);
                 notifyListeners();
             return true;

              }
              else
                  {
                      this.getStack(aIndex).push(from);
                      pop_from(from);
                      notifyListeners();
                  }
         }
         return true;
     }

 public void pop_from(CardStack to)
 {
     for(int j = 0 ; j < to.size() ; j ++)
         {

             this.getStack(fromIndex).pop(to.peek(j));

         }
         if(!this.getStack(fromIndex).peek().isFaceUp())
    {
        this.getStack(fromIndex).peek().setFaceUp(true);
    }

 }
 //如果发牌区全部发完并且discardstack也没有牌
 public void reset_all()
 {
     init();
     notifyListeners();
 }
 //如果发牌区全部发完  但是discardstack有牌
 public void reset()
 {
     for(int i = 0 ; i < this.getStack(1).size() ; i++)
     {
         this.getStack(0).init(this.getStack(1).peek(i));
     }

        this.getStack(1).clear();

     notifyListeners();
 }
 public CardStack StringToStack(String pString)
 {

    if(pString != null && pString.length() > 0)
    {
        String[] tokens = pString.split(SEPARATOR);
        CardStack aCards = new CardStack();
        for( int i = 0; i < tokens.length; i++ )
        {
            aCards.init(Card.get(tokens[i]));
        }
        for(int i = 0 ; i<aCards.size() ; i ++)
        {
            aCards.peek(i).setFaceUp(true);
        }
        return aCards;
    }
     return null;
 }
 public Card getTop(String result)
 {
     if( result != null && result.length() > 0)
     {
         String[] tokens = result.split(SEPARATOR);
         Card aCards [];
         aCards = new Card[tokens.length];
         for( int i = 0; i < tokens.length; i++ )
         {
             aCards[i] = Card.get(tokens[i]);
         }
         return aCards[0];
     }
     return null;
 }
 public String serialize(Card pCard, int aIndex)
 {

     CardStack temp_stack =  GameModel.instance().getSubStack(pCard, aIndex);
     String result = "";
     Card temp_card ;
     for(int i = 0 ; i< temp_stack.size() ; i++)
     {
         temp_card = temp_stack.peek(i);
         result += temp_card.getIDString()+SEPARATOR;
     }

     if( result.length() > 0)
     {
         result = result.substring(0, result.length()-1);
     }

     return result;
 }


 public void setFromIndex(int fromIndex) {
     this.fromIndex = fromIndex;
 }

 public void setLevel(String level)
 {
     this.level = level;
 }

```
**五．主要牌堆界面的实现类**
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020042709104259.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

首先定义了一个接口GameModelListenter 作为对于所有牌堆更新的监听器，里面定义了一个方法gameStateChanged，为更新牌堆的方法。
然后桌面堆，花色堆，发牌堆，丢弃堆都实现了这个接口，并且定义了方法 gameStateChanged。
这样做的用意是，在GameModel中定义一个泛型GameModelListenter为ArrayList，在每一个牌堆内，由于每个牌堆类都实现了GameModelListenter 的接口，所以可以将他们都加入到ArrayList中，通过 这种方式，如果要触发牌堆的更新，则遍历这个ArrayList中的每一个对象，然后调用他们的gameStateChanged方法更新界面，实现了界面的同步更新。
由于初始化ArrayList的每一个对象属于不同的类，定义了不同的gameStateChanged，调用时，通过多态的思想，会寻找到属于这个对象对应的gameStateChanged方法进行调用。

之后对于每一个界面类，都有自己界面的绘制方法，绘制方法都大致相同。
对于DescStack，将纸牌的背面初始化为按钮，为按钮设置点击事件，每次点击按钮，都会发一张牌到丢弃堆。

对于每一个牌堆的拖动的检测，共有五个回调函数。
createDragDetectedHandler（）当你在某个控件上移动时，会监测到拖动操作，执行这个函数。
createDragOverHandler（）当你拖动到目标上方时会执行。
createDragEnteredHandler（）当你进入目标控件时会执行
createDragExitedHandler（）当你移出一个目标控件时会执行
 createDragDroppedHandler（）当你移动到目标控件，并且松开鼠标时会执行

这五个函数之间配合，实现可卡牌的拖动逻辑。

TablePileView的实现如下，其他界面类的实现差不多：

```java
public class TablePileView extends StackPane implements GameModelListener
{
    private static final int PADDING = 5;
    private static final int Y_OFFSET = 17;
    private static final String SEPARATOR = ";";
    private static CardImages cardImages = new CardImages();
    private int aIndex;
    private static final String BORDER_STYLE = "-fx-border-color: lightgray;"
            + "-fx-border-width: 2.8;" + " -fx-border-radius: 10.0";
    TablePileView(TablePile pIndex)
    {
        aIndex = pIndex.ordinal()+2;//返回枚举类的序数  2-8
        setPadding(new Insets(PADDING));
        setStyle(BORDER_STYLE);
        setAlignment(Pos.TOP_CENTER);
        final ImageView image = new ImageView(cardImages.getBack());
        image.setVisible(false);
        getChildren().add(image);
        buildLayout();
        GameModel.instance().addListener(this);

    }
    private static Image getImage(Card pCard)
    {
            return cardImages.getCard(pCard);

    }
    private void buildLayout()
    {
        getChildren().clear();
        TableStack stack = (TableStack) GameModel.instance().getStack(aIndex);
        if( stack.isEmpty() )
        {
            ImageView image = new ImageView(cardImages.getBack());
            image.setVisible(false);
            getChildren().add(image);
            return;
        }
        for(int i = 0 ; i< stack.size() ; i ++)
        {
            Card cardView = stack.peek(i);
            if(cardView.isFaceUp() == true)
            { final ImageView image = new ImageView(getImage(cardView));
                image.setTranslateY(Y_OFFSET * i);
                getChildren().add(image);
                setOnDragOver(createDragOverHandler(image, cardView));//当你拖动到目标上方的时候，会不停的执行。
                setOnDragEntered(createDragEnteredHandler(image, cardView));// 当你拖动到目标控件的时候，会执行这个事件回调。
                setOnDragExited(createDragExitedHandler(image, cardView));//当你拖动移出目标控件的时候，执行这个操作。
                setOnDragDropped(createDragDroppedHandler(image, cardView));//当你拖动到目标并松开鼠标的时候，执行这个DragDropped事件。
                image.setOnDragDetected(createDragDetectedHandler(image, cardView));//当你从一个Node上进行拖动的时候，会检测到拖动操作，将会执行这个EventHandler。
            }
            else

            {
                final ImageView image = new ImageView(cardImages.getBack());
                image.setTranslateY(Y_OFFSET * i);
                getChildren().add(image);
            }


        }
    }

    private EventHandler<MouseEvent> createDragDetectedHandler(final ImageView pImageView, final Card pCard)
    {//当你从一个Node上进行拖动的时候，会检测到拖动操作，将会执行这个EventHandler。
        return new EventHandler<MouseEvent>()

        {
            @Override
            public void handle(MouseEvent pMouseEvent)
            {
                Dragboard db = pImageView.startDragAndDrop(TransferMode.ANY);
                ClipboardContent content = new ClipboardContent();
                content.putString(GameModel.instance().serialize(pCard,aIndex));
                db.setContent(content);
                pMouseEvent.consume();
                GameModel.instance().setFromIndex(aIndex);

            }
        };
    }

    private EventHandler<DragEvent> createDragOverHandler(final ImageView pImageView, final Card pCard)
    {
        //当你拖动到目标上方的时候，会不停的执行。
        return new EventHandler<DragEvent>()
        {
            @Override
            public void handle(DragEvent pEvent)
            {
                if(pEvent.getGestureSource() != pImageView && pEvent.getDragboard().hasString())
                {

                    if( GameModel.instance().isLegalMove(GameModel.instance().getTop(pEvent.getDragboard().getString()), aIndex) )
                    {
                        pEvent.acceptTransferModes(TransferMode.MOVE);
                    }
                }
                pEvent.consume();
            }
        };
    }

    private EventHandler<DragEvent> createDragEnteredHandler(final ImageView pImageView, final Card pCard)
    {
        // 当你拖动到目标控件的时候，会执行这个事件回调。
        return new EventHandler<DragEvent>()
        {
            @Override
            public void handle(DragEvent pEvent)
            {
                if( GameModel.instance().isLegalMove(GameModel.instance().getTop(pEvent.getDragboard().getString()), aIndex) )
                {
                    pImageView.setEffect(new DropShadow());


                }
                pEvent.consume();
            }
        };
    }

    private EventHandler<DragEvent> createDragExitedHandler(final ImageView pImageView, final Card pCard)
    {
        //当你拖动移出目标控件的时候，执行这个操作。
        return new EventHandler<DragEvent>()
        {
            @Override
            public void handle(DragEvent pEvent)
            {
                pImageView.setEffect(null);
                pEvent.consume();
            }
        };
    }

    private EventHandler<DragEvent> createDragDroppedHandler(final ImageView pImageView, final Card pCard)
    {
        //当你拖动到目标并松开鼠标的时候，执行这个DragDropped事件。
        return new EventHandler<DragEvent>()
        {
            @Override
            public void handle(DragEvent pEvent)
            {
                Dragboard db = pEvent.getDragboard();
                boolean success = false;
                if(db.hasString())
                {
                   boolean a =  GameModel.instance().moveCard(GameModel.instance().StringToStack(db.getString()), aIndex);
                    System.out.println(a);
                    success = true;
                    ClipboardContent content = new ClipboardContent();
                    content.putString(null);
                    db.setContent(content);
                    }
                pEvent.setDropCompleted(success);
                pEvent.consume();
            }
        };
    }
    //获取移动时最顶端的卡片

    public  void gameStateChanged()
    {
        buildLayout();
    }
}
```

**悔牌的逻辑；将每一步的操作保存到堆栈中，然后每次点击一次悔牌按钮，都让一个arraylist出栈，然后实现了悔牌功能。**

```java
if(!listStack.isEmpty())
    {
        ArrayList<CardStack> temp_list = listStack.peek();
        listStack.pop();
        ArrayList<Integer> flag = new ArrayList<Integer>();
        for (int i = 2 ; i<9 ; i++)
        {
            if(this.card_Stacks.get(i).size() != temp_list.get(i).size())
            {
                flag.add(i);
            }
        }
        if(flag.size() == 1)
        {
            if(this.card_Stacks.get(flag.get(0)).size() <temp_list.get(flag.get(0)).size() &&this.card_Stacks.get(flag.get(0)).size() >0 && temp_list.get(flag.get(0)).size() > 1)
            {
                if(temp_list.get(flag.get(0)).peek(temp_list.get(flag.get(0)).size() - 2).isFaceUp())
                {
                    temp_list.get(flag.get(0)).peek(temp_list.get(flag.get(0)).size()-2).setFaceUp(false);
                }
            }

        }
        if(flag.size() ==2)
        {
            if(this.card_Stacks.get(flag.get(0)).size() -temp_list.get(flag.get(0)).size() == 1 )
            {

                temp_list.get(flag.get(1)).peek(temp_list.get(flag.get(1)).size()-2).setFaceUp(false);

            }
            if(this.card_Stacks.get(flag.get(1)).size() -temp_list.get(flag.get(1)).size()  ==1)
            {
                temp_list.get(flag.get(0)).peek(temp_list.get(flag.get(0)).size()-2).setFaceUp(false);
            }
            if(this.card_Stacks.get(flag.get(0)).size() - temp_list.get(flag.get(0)).size() >=2)
            {
                int i = this.card_Stacks.get(flag.get(0)).size() - temp_list.get(flag.get(0)).size();
                temp_list.get(flag.get(1)).peek(temp_list.get(flag.get(1)).size()-i-1).setFaceUp(false);

            }
            if(this.card_Stacks.get(flag.get(1)).size() -temp_list.get(flag.get(1)).size() >=2)
            {
                int i = this.card_Stacks.get(flag.get(1)).size() - temp_list.get(flag.get(1)).size();
                temp_list.get(flag.get(0)).peek(temp_list.get(flag.get(0)).size()-i-1).setFaceUp(false);

            }

        }



        for(int i = 0 ; i< 13 ; i++)
        {
            this.card_Stacks.get(i).clear();
        }
        for(int i = 0 ; i< 13 ; i++)
        {
            for(int j = 0 ; j <temp_list.get(i).size() ; j++)
            {

                this.card_Stacks.get(i).init(temp_list.get(i).peek(j));
            }
        }
        notifyListeners();
    }
    else
    {
        return;
    }
```
## 结果展示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427091202110.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427091207478.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427091210768.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
