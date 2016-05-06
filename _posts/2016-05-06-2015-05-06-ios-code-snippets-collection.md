---
layout: blog
categories: blog
tags: 
  - iOS
published: true
title: iOS 常用代码集合
---
## UIView和UIViewController相关代码集合


1. 通过code加载view controller

		ViewController *view1 = [storyBaord instantiateViewControllerWithIdentifier:@"helloview"];

2. 通过tag获取view

		UIView *view = [incomingViewController.view  viewWithTag:1];

3. 添加subview，index为0代表添加到最底层。

		[view insertSubview:segmentedControl atIndex:0];

4. 把一个ui view从父view中删除

		[viewController.view removeFromSuperview];

5. 切换view动画

       [UIView beginAnimations:@"View flip" context:nil];
       [UIView setAnimationDuration:0.4];
       [UIView setAnimationCurve:UIViewAnimationCurveEaseInOut];
       [UIView setAnimationTransition:UIViewAnimationTransitionFlipFromLeft forView:self.view cache:YES];
       [UIView commitAnimations];

6. 通过代码初始化UIView，先创建一个Rect区域用于显示uiview，然后调用initWithFrame

       CGRect rect = CGRectMake(0, 0, 80, 80); 
       UILabel *label = [[UILabel alloc] initWithFrame:rect];

7. 获取当前执行的方法的签名：

       NSString *signature = NSStringFromSelector(_cmd);

8. 检测当前手机Orientation

        UIInterfaceOrientation orientation = [UIApplication sharedApplication].statusBarOrientation;
        switch (orientation) {
            case UIInterfaceOrientationPortrait:
                size.width = [UIScreen mainScreen].applicationFrame.size.width / 3;
                break;
            case UIInterfaceOrientationLandscapeLeft:
            case UIInterfaceOrientationLandscapeRight:
                size.width = [UIScreen mainScreen].applicationFrame.size.height / 5;
                break;
            default:
                size.width = [UIScreen mainScreen].applicationFrame.size.width / 3;
                break;
        }

9. 渐变颜色

        UIView *view = [[UIView alloc] initWithFrame:CGRectMake(0.0f, 0.0f, 320.0f, 50.0f)];
        CAGradientLayer *gradient = [CAGradientLayer layer];
        gradient.frame = view.bounds;
        gradient.colors = [NSArray arrayWithObjects:(id)[[UIColor whiteColor] CGColor], (id)[[UIColor blackColor] CGColor], nil];
        [view.layer insertSublayer:gradient atIndex:0];



## UIButton

1. 触发UIButton click event

		[button sendActionsForControlEvents:UIControlEventTouchUpInside];


## NSString 格式化



## 字体UIFont

1. 创建字体

		UIFont *font = [UIFont fontWithName:@"Arial" size:14];

2. 计算字体高度

		font.ascender - font.descender

3. 获取默认字体

		UIFont *defaultFont = [UIFont preferredFontForTextStyle:UIFontTextStyleBody];

4. 设置字体样式

       NSMutableParagraphStyle *style = [[NSMutableParagraphStyle alloc] init];
       [style setLineBreakMode:NSLineBreakByCharWrapping];
       NSDictionary *attr = @{ NSFontAttributeName: font, NSParagraphStyleAttributeName: style };
       [string boundingRectWithSize:maxSize options:opts attributes:attrs context:nil];


## UIPicker

1. 建立依赖关系的picker

a) picker两个component分别绑定数据源
b) 继承UIPickerViewDelegate，并实现picker: didSelectRow: inComponent 方法
c) 判断是哪个component发生变化，如果是主component发生变化，则让第二个的component的datasource也随之变化。

## UITableView 

建立table view流程

a) 通过tag获取 table view    

	UITableView *tableView = (id)[self.view viewWithTag:1]
          
b) 或者通过IBOutlet    

c) 实现tableView: numberOfRowsInSection方法，返回每个section有多少行    

d) 实现tableView: cellForRowAtIndexPath，返回每个行对应的cell，为了重用UITableViewCell，而无需重复的新建或释放cell，可以使用`[tableView dequeueReusableCellWithIdentifier]`    

e) 若需要给table view cell添加image view，可以使用：`cell.imageView.image = <UIImage instance>`    

f) UITableViewCell style 分为4种： `UITableViewCellStyleDefault` (Text label), `UITableViewCellStyleSubtitle` (Detail), `UITableViewCellStyleValue1` (text on left bold, and small text on right with blue), `UITableViewCellStyleValue2` (phone book style)    

g) Indent level:  `tableView: indentationLevelForRowAtIndexPath`    

h) 选择某行：`tableView: willSelectRowAtIndexPath`会在选中某行前被调用，可以在这个方法中定义哪些行可以被选中，而哪些不可以; `tableView: didSelectRowAtIndexPath`是在某行被选中后调用。    

i) 改变字体：`cell.textLabel.font = <UIFont>; cell.detailTextLabel.text = ...`    

j) 改变行高度： `tableView: heightForRowAtIndexPath`    

k) 添加自定义的table view cell: 首先创建一个继承于UITableViewCell的类，然后调用UITableView的实例方法：`registerClass: [YouTableViewCell class] forCellReuseIdentifier: “cellIdentifier"`    

l) 从nib加载UITableViewCell：

	UINib *nib = [UINibnibWithNibName:@"nibResouceName"bundle:nil];
	[tableView registerNib: nib forCellReuseIdentifier:@"cellIdentifier"];
        
m) 使用section：添加：tableView: titleForHeaderInSection:方法，用于显示当前section的名字；indexPath.section － 当前的section，indexPath.row - 当前的row index    

n) 添加index：添加方法，`- (NSArray *)sectionIndexTitlesForTableView:(UITableView *)tableView`, 并返回index的集合即可。    

o) 添加搜索框：首先，遵循 UISearchDisplayDelegate协议，增加两个成员，一个是搜索的结果NSMutableArray, 一个是`UISearchDisplayController`, 创建`UISearchBar`, 并添加到`tableView.tableHeaderView = searchBar`； 初始化一个`UISearchBarController`, 并指定`delegate`； 实现`searchDisplayController: didLoadSearchResultsTableView`，并注册table view cell； 为每一个原来的delegate方法添加搜索结果的数据源；最后是搜索的逻辑：`searchBarController: shouldReloadTableForSearchString:`     

p) 刷新table view: `[self.tableView reloadData]`;    

q) table view增加偏移，不与status bar重叠

	UIEdgeInsets contentInset = self.tableView.contentInset;
	contentInset.top = 20;
	[self.tableView setContentInset: contentInset];
          

## Collection view

1. 创建CollectionViewController

2. 创建CollectionViewCell

3. 注册custom collection view cell (viewDidLoad 方法内添加)

		[vc.collectionView registerClass:[YourCollectionViewCell class] forCellWithReuseIdentifier:@"Content"]


## Navigation Controller

1. 切换scene

a) ctrl-Drag 拖动cell至另外一个view，选择push，创建一个segue.    

b) 实现prepareForSegue: sender方法，`UIViewController *vc = segue.destinationViewController`获取目标view controller。    
               
2. 添加accessory action: ctrl-drag, 拖动cell至另外一个view, 选择action accessory，创建info segue    

3. 区分不同segue: 给每一个segue添加segue identifier    

4. 创建swipe-to-delete功能    

a) 添加table view方法：tableView: canEditRowAtIndexPath: ，返回true则表示该行可以编辑。    
b) tableView: commitEditingStyle: forRowAtIndexPath:     
c) 在上面的方法内，加入删除逻辑，并在最后调用：[tableView deleteRowAtIndexPaths: withRowAnimation:]删除该行    

5. 创建drag-to-reorder效果

a) 添加编辑按钮, `self.navigationItem.rightBarButtonItem = self.editButtonitem`    

b) 实现table View delegate方法：tableView: `moveRowAtIndexPath: toIndexPath`, `fromIndexPath`是移动前的位置，`toIndexPath`是移动后的位置。    

6. 设置navigation bar 背景颜色

		[[self navigationBar] setBarTintColor:[UIColor colorWithHex:kNaviBarBackgroundColor alpha:1.0]];

或者全局设置

		[[UINavigationBar appearance] setBarTintColor:[UIColor whiteColor]];  

7. 设置navigation bar button colour 

		[[self navigationBar] setTintColor:[UIColor whiteColor]];


## 加载资源和User Default

1. 加载plist资源

       NSBundle *bundle = [NSBundle mainBundle];
       NSURL *plistURL = [bundle URLForResource:@"filename" withExtension:@"plist"];
       NSDictionary *dict = [NSDictionary dictionaryWithContentsOfURL:plistURL];

2. 从Nib中加载UIView

	UINib *nib = [UINib nibWithNibName:@"nibResouceName" bundle:nil];

3. 保存数据到User Default

	NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
	[defaults setObject:objs forKey:@"your object"];
	[defaults synchronize];

4. 读取User default的数据

	NSUserDefaults *defaults  = [NSUserDefaults standardUserDefaults];
	NSArray *data = [defaults objectForKey:@"your data key"];
    self.list = [data mutableCopy];

5. Application setting bundle

a) 创建Root.plist (File > New > Resource > setting bundle)

b) PeferenceSpecifiers增加item

c) 展开item 0，修改title和type, type为PSGroupSpecifier, 因为至少要有一个group

d) 接下来添加item 1/2/3..., item 1/2/3… 为item 0这个group的属性。它们的type可以使PSTextfieldSpecifier (文本输入框), PSMultiValueSpecifier (单选框), PSToggleSwitchSpecifier (二值选项，YES/NO), PSGroupSpecifier (min － max slider), PSChildPaneSpecifier (Drill down Subview, 指定另外一个plist文件为设置选项)

e) 注册user default, 获取未曾设置的默认值

		NSDictionary *defaults = @{key: value};
		[[NSUserDefaults standardUserDefaults] registerDefaults:defaults];
          
f) 读取setting bundle, 使用NSUserDefaults读取。

g) 修改user defaults: [defaults setBool: forKey:], [defaults setFloat forKey:], ...
 
6. 获取app的document dir, NSDocumentationDirectory：搜索Document dir, NSUserDomainMask：在app的sandbox搜索。

		NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentationDirectory, NSUserDomainMask, YES);
		NSString *documentPath = paths[0];

7. 获取document文件夹某个文件的绝对路径：

		NSString *filePath = [documentPath stringByAppendingPathComponent:@"hello.txt"];

8. 获取临时文件夹

		NSString *tempPath = NSTemporaryDirectory();

10. 检查文件是否存在

		if([[NSFileManager defaultManager] fileExistsAtPath:filePath]){}


12. 支持对象复制（遵循NSCopying协议）, 实现copyWithZone:方法，复制每一个属性

13. 获取文件夹里面的文件列表
     
        NSFileManager *fileManager = [NSFileManager defaultManager];
        NSString *motionsDir = [[[NSBundle mainBundle] bundlePath] stringByAppendingPathComponent:kMotionFileDir];
        NSArray *motionsContents = [fileManager contentsOfDirectoryAtPath:motionsDir error:nil];
        NSPredicate *filter = [NSPredicate predicateWithFormat:@"self STARTSWITH '.plist'"];
        NSArray *motionFiles = [motionsContents filteredArrayUsingPredicate:filter];


## 存储数据

1. property list

a) Property list保存数据

可以支持序列化的对象类型： NSArray, NSDictionary, NSData, NSString, NSNumber, NSDate。

序列化对象调用writeToFile保存数据：`[array writeToFile:@“/a/b/c.plist”, atomically: YES]`; 或者，`writeToURL:atomically`:

b) 读取property list

       NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentationDirectory, NSUserDomainMask, YES);
       NSString *filePath = [paths[0] stringByAppendingPathComponent:@"hello.plist"];
       if([[NSFileManager defaultManager] fileExistsAtPath:filePath]){
          NSArray *array = [[NSArray alloc] initWithContentsOfFile:filePath];
          self.data1 = array[0];
          self.data2 = array[1];
       }

2. object archives

a) object必须遵循NSCoding协议，并实现两个协议方法：

		- (void)encodeWithCoder:(NSCoder *)aCoder;
		- (nullable instancetype)initWithCoder:(NSCoder *)aDecoder;
          
b) 对于encodeWithCoder, 调用参数NSCoder *encoder的方法，序列化每一个属性
c) 对于initWithCoder方法，调用decoder 的decodeXXForKey来载入每一个属性
d) 保存archive object：首先创建 NSMutableData用于保存对象数据，然后调用archiver encode对象，最后保存至文件

		NSMutableData *data = [[NSMutableData alloc] init];
		NSKeyedArchiver *archiver = [[NSKeyedArchiver alloc] initForWritingWithMutableData:data];
		[archiver encodeObject:myObject forKey:@"keyValueString"];
		[archiver finishEncoding];
		BOOL success = [data writeToFile:@"/Path" atomically:YES];

e) 读取archive object: 首先从文件载入NSdata，然后创建NSKeyedUnarchiver并反序列化对象，最后告诉unarchiver结束decoding:

		NSData *data = [[NSData alloc] initWithContentsOfFile:@"/Path"];
		NSKeyedUnarchiver *unarchiver = [[NSKeyedUnarchiver alloc] initForReadingWithData:data];
		self.object = [unarchiver decodeObjectForKey:@"keyValueString"];
		[unarchiver finishDecoding];

3. SQLite3

a) 打开数据库

		#import <sqlite3.h>
		sqlite3 *db;
		sqlite3_open("path", &db);
          
b) 关闭数据库

		sqlite3_close(db);

c) 执行select 语句

        NSString *sql = @"select name from person";
        sqlite3_stmt *stmt;
        int result = sqlite3_prepare_v2(db, [sql UTF8String], -1, &stmt, nil);
        while (sqlite3_step(stmt)) {
            int rowNum = sqlite3_column_int(stmt, 0);
            char *rowData = sqlite3_column_text(stmt, 1);
            NSString *fieldValue = [[NSString alloc] initWithUTF8String:rowData];

        }
        sqlite3_finalize(stmt);

d) 执行non select语句

        char *sql = "insert into abc values(?, ?)";
        if(sqlite3_prepare_v2(db, sql, -1, &stmt, nil) == SQLITE_OK){
            sqlite3_bind_int(stmt, 1, 123);
            sqlite3_bind_text(stmt, 2, "hi", -1, NULL);
        }
        if(sqlite3_step(stmt)) {
            // Error
        }
        sqlite3_finalize(stmt);

4. Core Data

a) 创建项目，并勾选use core data

b) 打开文件xx.xcdatamodeld文件，启动data model editor

c) 创建attributes, relationships, fetch properties

d) 创建managed object

		NSManagedObject *obj = [NSEntityDescription insertNewObjectForEntityForName:@"Person" inManagedObjectContext:context];
          
e) 获取managed object

		AppDelegate *app = [[UIApplication sharedApplication] delegate];
		NSManagedObjectContext *context = [app managedObjectContext];
		NSFetchRequest *request = [[NSFetchRequest alloc] init];
		NSEntityDescription *desc = [NSEntityDescription entityForName:@"Person" inManagedObjectContext:context];
		[request setEntity:desc];
        
f) 条件查询 （predicate）

		NSPredicate *predicate = [NSPredicate predicateWithFormat:@"(name = @%)", name];
		[request setPredicate:predicate];

g) 执行request查询

    	NSError *error;
    	NSArray *objects = [context executeFetchRequest:request error:&error];
        
h) 获取object的attribute （使用key-value-coding)

		NSString *name = [objects[0] valueForKey:@"name"];
		NSNumber *age = [objects[0] valueForKey:@"age"];

i) 修改object的attribute （使用kvc)

		[objects[0] setValue:@"enix" forKey:@"name"];
		[objects[0] setValue:@18 forKey:@"age"];
		[app saveContext];

5. Realm-cocoa


## 多线程 (GCD)

1. Queue, group, 异步执行


a) 创建queue，并添加step至queue

    	dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    	dispatch_async(queue, ^{
        	//task
   		 });

b) 异步执行task
    
    	dispatch_group_t group = dispatch_group_create();
        dispatch_group_async(group, queue, ^{
            //task 1;
        });
        dispatch_group_async(group, queue, ^{
            //task 2;
        });
        dispatch_group_notify(group, queue, ^{
            // run when all the task finished in group
        })

2. Block

a) 创建block

          void (^hello)(void) = ^{ NSLog(@"I am a block "); };
          hello();
          
b) block 在创建的地方复制一份当前上下文的变量的值；如果需要改变block以外的变量的值，需要在声明变量的地方加上__


## 正则表达式 (NSRegularExpression)

1. 测试字符串

		NSString *pattern = @"^#[0-9a-fA-F]{6}$";
		NSError *error;
		NSRegularExpression *regex = [NSRegularExpression regularExpressionWithPattern:pattern options:0 error:&error];
		if(![regex numberOfMatchesInString:str options:0 range:NSMakeRange(0, [str length])]){
			NSLog(@"Invalid str [%@]!", str);
		}

2. 获取匹配的group


## 日期相关 (NSDate)

1. 获取当前日期：
     
		NSDate *now = [NSDate date];

2. 格式化输出：

       NSDate *now = [NSDate date];
       NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
       [formatter setDateFormat:@"yyyy-MM-dd HH:mm:ss"];
       NSString *dateStr = [formatter stringFromDate:now];


## 网络相关

1. 网络请求

2. 中文URL encode

判断系统版本

    /*
     *  System Versioning Preprocessor Macros
     */ 

    #define SYSTEM_VERSION_EQUAL_TO(v)                  ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] == NSOrderedSame)
    #define SYSTEM_VERSION_GREATER_THAN(v)              ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] == NSOrderedDescending)
    #define SYSTEM_VERSION_GREATER_THAN_OR_EQUAL_TO(v)  ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] != NSOrderedAscending)
    #define SYSTEM_VERSION_LESS_THAN(v)                 ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] == NSOrderedAscending)
    #define SYSTEM_VERSION_LESS_THAN_OR_EQUAL_TO(v)     ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] != NSOrderedDescending)

    /*
     *  Usage
     */ 

    if (SYSTEM_VERSION_LESS_THAN(@"4.0")) {
        ...
    }

    if (SYSTEM_VERSION_GREATER_THAN_OR_EQUAL_TO(@"3.1.1")) {
        ...
    }


Design pattern

1. 单例模式（singleton）

通过dispatch_once_t实现初始化一次实例的逻辑。

    + (instancetype)shareInstance{
        static YourClass *instance = nil;
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken, ^{
            instance = [[self alloc] init];
        });
        return instance;
    }

2. Immutable property define in header, while define the same property with mutable type

因为基本所有的mutable class都是对应的immutable class的子类，所以我们暴露给外面的属性使用immutable，阻止调用程序误改了数据，而在类的内部使用mutable，是方便内部修改数据。这样可以实现了很好的数据封装。

header 

		@interface:
		- (NSArray *)list;

implementation:

		@property (nonatomic, strong)NSMutableArray *list;

3. Notification center观察者模式

notification center是一个单例，可用于类之间的通知。例如，注册UIApplication的`UIApplicationWillEnterForegroundNotification`事件的观察者为self，并在事件发生时，调用self的yourMethod方法。

		UIApplication *app = [UIApplication sharedApplication];
		[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(yourMethod) name:UIApplicationWillEnterForegroundNotification object:app]

4. Key-value coding

KVC不直接调用object的方法，而是通过valueForKey, 查询并访问对应的属性。使用`setValue: forKey:`修改属性。


## NSData 

1. NSData 转换为NSString

		NSString *myString = [[NSString alloc] initWithData:myData encoding:NSUTF8StringEncoding];

2. NSData格式化为HEX NSString



## UITextView / UITextField

1. text view滚动至底部

		[TextView scrollRangeToVisible:NSMakeRange([TextView.text length], 0)];


## i18n国际化

1. Get the current user language

		NSString * language = [[NSLocale preferredLanguages] objectAtIndex:0];

## Quartz 2D

1. Path Gradient

        // Get current context
        CGContextRef context = UIGraphicsGetCurrentContext();

        // Save current context
        CGContextSaveGState(context);
        // Add a path to the context
        CGContextAddPath(context, bezierPath.CGPath);

        // Intersect the context's path with the current clip path
        CGContextClip(context);

        // Create RGB Color space
        colorSpace = CGColorSpaceCreateDeviceRGB();

        // Create gradient colors
        NSArray *colors;
        if (_touchDown) {
            colors = [NSArray arrayWithObjects:(id)[UIColor colorWithRed:0.96 green:0.96 blue:0.96 alpha:1.0].CGColor ,(id)[UIColor colorWithRed:0.842 green:0.842 blue:0.842 alpha:1.0].CGColor, nil];
        } else {
            colors = [NSArray arrayWithObjects:(id)[UIColor colorWithRed:0.862 green:0.862 blue:0.862 alpha:1.0].CGColor, (id)[UIColor colorWithRed:0.96 green:0.96 blue:0.96 alpha:1.0].CGColor, nil];
        }

        // Create Gradient
        CGGradientRef gradient = CGGradientCreateWithColors(colorSpace, (CFArrayRef)colors, NULL);

        // Draw gradient
        CGContextDrawLinearGradient(context, gradient, rect.origin, curPoint, kNilOptions);

        // Decrese the colorspace retain count
        CGColorSpaceRelease(colorSpace);
        colorSpace = NULL;

        // Restore the context
        CGContextRestoreGState(context);

2. Shadow

        // Get current context
        CGContextRef context = UIGraphicsGetCurrentContext();

        // Save current context
        CGContextSaveGState(context);

        // Draw a shadow
        CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
        CGSize shadowOffset = CGSizeMake(1, 3);
        CGFloat shadowColor[] = {225/255.0, 225/255.0, 225/255.0, 0.6};
        CGColorRef shadowColorRef = CGColorCreate(colorSpace, shadowColor);
        CGContextSetShadowWithColor(context, shadowOffset, 3, shadowColorRef);

        CGColorRelease(shadowColorRef);
        CGColorSpaceRelease(colorSpace);
        colorSpace = NULL;

        // Restore the context
        CGContextRestoreGState(context);

3. Shadow for UIView

        self.layer.masksToBounds = NO;
        self.layer.shadowOffset = CGSizeMake(-15, 20);
        self.layer.shadowRadius = 5;
        self.layer.shadowOpacity = 0.5;

## Autolayout issues

Trailing space -20 issues

http://stackoverflow.com/questions/26447266/ios-autolayout-changes-leading-and-trailing-space/26729064
