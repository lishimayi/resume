# iOS View的声明周期（即ViewController的声明周期）

```

// 1
- (instancetype)init;
- (instancetype)initWithCoder:(NSCoder *)coder;
- (instancetype)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil ;
// 2
- (void)loadView;
// 3
- (void)viewDidLoad;
 // 4
 - (void)viewWillAppear:(BOOL)animated

 //5
 - (void)viewWillLayoutSubviews;

 //6
 - (void)viewDidLayoutSubviews;

 //7
 - (void)viewDidAppear:(BOOL)animated;

 //8
 - (void)viewWillDisappear:(BOOL)animated;

 //9
 - (void)viewDidDisappear:(BOOL)animated;

//10
 - (void)viewWillUnload;

//11
- (void)viewDidUnload;

```