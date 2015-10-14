---
layout: post
title: "一种更简单的UITableView使用方式"
date: 2015-10-08 20:57
comments: true
categories: 
  - iOS
---
UITableView是iOS开发中非常常用的一个UI组件，然而传统的使用方式有点乏味，通常的写法是：

* 将数据源放入一个数组中
* 围绕这个数组定义UITableDataSource里的各个方法
* 如果高度有变化则还需要实现UITableViewDelegate里的对应方法
* 当你的数组里有多种类型的item时，上面所述的方法不可避免的包含一大堆ifelse

当一遍又一遍地重复写这些boilerplate代码时，日子就会变得很枯燥。有没有更加轻便点的写法呢？

objc.io早在issue 1里其实就提到了一个[思路](https://www.objc.io/issues/1-view-controllers/table-views/#bridging-the-gap-between-model-objects-and-cells)，于是我将它扩展了下，变成了一个支持多类型Cell的可重用的数据源([Github](https://github.com/perrywky/CCTableDataSource))。它封装了一个二维数组，实现了UITableViewDataSource和UITableViewDelegate的那些无聊琐屑的方法，只暴露了几个添加数据的接口，使用起来会轻松得多。废话不多说，看下面一段示例代码：

``` objective-c

/*
 * 这是一段展示用户头像和照片流的代码，类似Instagram
 */
  
  NSArray *dataArray; //假设封装前的数据已都放入这个数组
  CCTableDataSource *ds = [[CCTableDataSource alloc] initWithTableView:tableView];

  CCTableComponent *seperator = [CCTableComponent componentWithCellConfigure:^(UITableViewCell *cell) {
    //特殊的分割样式
  } andHeight:8.5];

  for (User *user in dataArray) {//假设数组里都是User对象
    NSUInteger section = [tableDataSource addSection]; //每个用户一个Section
    CCTableComponent *userHeader = [CCTableComponent componentWithClass:[UserHeader class]//UserHeader继承自UITableViewHeaderFooterView 
                                                               data:user 
                                                         identifier:@"user"];
    [tableDataSource setHeader:userHeader ofSection:section];//用户头像作为SectionHeader

    for (Photo *photo in user.photos) {
      CCTableComponent *photoCell = [CCTableComponent componentWithClass:[PhotoCell class] 
                                                                data:photo 
                                                          identifier:@"photo"
                                                       selectedBlock:^(NSIndexPath *indexPath, 
                                                                       UITableViewCell<CCTableComponentDelegate> *cell, 
                                                                       CCTableComponent *component) {
        //show large photo
      }];
      [self.tableDataSource addCell:photoCell toSection:section];
    }

    [self.tableDataSource addCell:seperator toSection:section];
  }
```

这段代码里使用了三种UI控件，分别是UserHeader、PhotoCell和SeperatorCell，如果采用传统的方式，那么代码会变得有点冗长...

``` objective-c
/*
 * 假设数据都已放入self.dataArray
 */

-(void)initTable
{
  [self.tableView registerClass:[UserHeader class] forHeaderFooterViewReuseIdentifier:@"user"];
  [self.tableView registerClass:[PhotoCell class] forCellReuseIdentifier:@"photo"];
  [self.tableView registerClass:[UITableViewCell class] forCellReuseIdentifier:@"seperator"];
}

- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView
{
  return self.dataArray.count;
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
  return [(User *)self.dataArray[section] photos].count + 1; //photos + seperator
}

-(UIView *)tableView:(UITableView *)tableView viewForHeaderInSection:(NSInteger)section
{
  UserHeader *header = (UserHeader *)[tableView dequeueReusableHeaderFooterViewWithIdentifier:@"user"];
  header.user = self.data[section];
  return header;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
  NSArray *photos = [(User *)self.dataArray[indexPath.section] photos];
  if (indexPath.row < photos.count) {
      PhotoCell *cell = (PhotoCell *)[tableView dequeueReusableCellWithIdentifier:@"photo"];
      cell.photo = photos[indexPath.row];
      return cell;
  } else {
      UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:@"seperator"];
      //configure for the first time
      return cell;
  }
}

-(CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath
{
  NSArray *photos = [(User *)self.dataArray[indexPath.section] photos];
  if (indexPath.row < photos.count) {
    //return photo cell height
  } else {
    return 8.5;
  }
}

-(void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath
{
  NSArray *photos = [(User *)self.dataArray[indexPath.section] photos];
  if (indexPaht.row < photos.count) {
    Photo *photo = photos[indexPath.row];
    //show large photo
  }
}

```

注意为了便于阅读，第一段代码里我使用了折行，实际代码调用量的对比更加明显。而且随着业务的发展，我们很可能会增加Cell的种类。

* 比如中间插条广告？
* 推荐些达人？
* 再展示点最新评论？
* 当然还有必须的加载更多。。。

当这些都凑齐后，可想而知第二段代码会变得多么冗长，每个和section、cell相关的方法里都会塞满一堆ifelse判断。。。而第一段代码则依然保持优雅，你只需要按照数据源遍历一次，将各种类型的section或cell按顺序加进CCTableDataSource，剩下的工作就全交给它啦。

当然，想要实现这样的效果，光靠一个CCTableDataSource是不够的，你需要将自己定义的Cell或SectionHeaderFooter实现以下Protocol

``` objective-c
@protocol CCTableComponentDelegate<NSObject>

-(void)configureWithData:(id)data; //你对UI的设置代码需要放在这里面
+(CGFloat)heightForData:(id)data; //给CCTableDataSource用来设置高度

@end
```

然后将其用一个CCTableComponent封装起来，CCTableDataSource里维护的就是一个二维的CCTableComponent数组，不管是Cell还是SectionHeaderFooter，都视为一个component。

``` objective-c
@interface CCTableComponent : NSObject

@property (nonatomic, strong) Class<CCTableComponentDelegate> componentClass; //实际需要显示的UI Class
@property (nonatomic, strong) id data; //用于显示的数据
@property (nonatomic, strong) NSString *componentIdentifier; //重用的标识符
@property (nonatomic, strong) CCSelectCellBlock selectCellBlock; //选中后执行的block(不是必须)

@end
```

如果你需要使用TableViewDelegate里定义的其他方法，可以把定义的delegate传进来

``` objective-c
-(id)initWithTableView:(UITableView *)tableview delegate:(id<UITableViewDelegate>)delegate;
```

CCTableDataSource会负责转发，但是要注意的是，我还没有实现所有的方法，如果需要你可以自行添加。

如果你的TableView结构简单，这么做可能没啥必要，但一旦你习惯这种写法后，当TableView内容开始复杂时会感觉愉悦不少，赶紧试试吧~

Github地址：[https://github.com/perrywky/CCTableDataSource](https://github.com/perrywky/CCTableDataSource)
