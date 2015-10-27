# 我的日程

## 我的日程结构图

![我的日程](我的日程2.png)


我的日程展示当前用户的所有日程，包括**新日程**<未完成的日程>,**历史日程**,展示时按照时间排序<降序>.



我的日程效果图：


![我的日程记录](我的日程记录1.png)



```swift
//展示我的日程准备工作

- (void)viewDidLoad
{
    [super viewDidLoad];

    self.title = @"我的日程";

    
    //我的日程分两部分，第一部分为“新日程”，第二部分为”历史日程“

    nowScheduleArr = [[NSMutableArray alloc]init];
    historyScheduleArr = [[NSMutableArray alloc]init];

    //获取当前用户的登录信息

    userModel = [LoginUserModel getCurrentLoginInfo].usermodel;
    
    userModel.user_scheduleArr = [[NSMutableArray alloc]init]; //按月分组后的数组
    
    userModel.user_scheduleHistoryArr = [[NSMutableArray alloc]init]; //历史日程
    
    
    //创建一个显示所有“我的日程”数据

    myTable = [[UITableView alloc]initWithFrame:CGRectMake(0, 0, UIWidth, UIHeight-UI_NavY-UI_Bar) style:UITableViewStyleGrouped];
    
    //myTable的分割线样式设为None
    
    myTable.separatorStyle = UITableViewCellSeparatorStyleNone;
    myTable.backgroundColor = [UIColor clearColor];
    
    //设置myTable的代理对象
    
    myTable.delegate = self;
    myTable.dataSource = self;
    
    //将myTable添加到MySchViewController视图控制器上
    
    [self.view addSubview:myTable];
    
    
    //设置“数字”通知角标
    
    if(ISIOS7)
    {
        self.edgesForExtendedLayout = UIRectEdgeNone;
    }
    
    //懒加载刷新控件

	if (_refreshHeaderView == nil) {

		EGORefreshTableHeaderView *view = [[EGORefreshTableHeaderView alloc] initWithFrame:CGRectMake(0.0f, -200.0f, myTable.frame.size.width, 200.0f)];
        
        //设置EGORefreshTableHeaderView的代理对象
        
		view.delegate = self;
        view.backgroundColor = [UIColor clearColor];
        
        //将EGORefreshTableHeaderView添加到tableView上
        
		[myTable addSubview:view];
		_refreshHeaderView = view;
		[view release];

	}
    
    //加载所有的日程数据

    [self getDataSource];


    //注册一个监听添加日程的通知（添加日程后，当前界面要重新获取数据并刷新）
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(addNewSchedule:) name:AddScheduleNoti object:nil];

    //添加一个删除提醒的通知
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(getDataSource) name:DeleScheduleNoti object:nil];

    //监听“我的来访通知”系统通知-----已作废
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(backToSelf) name:NotiComeFri object:nil];
    
    //监听点击单个记录的通知
    
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(gotoSchDetail:) name:NOtiOneSchedule object:nil];
    
    //设置tableView的footerView

    footView = [[LoadMoreView alloc]initWithFrame:CGRectMake(0, 0, UIWidth, 40)];
    footView.backgroundColor = [UIColor clearColor];
    footView.hidden = YES;
    myTable.tableFooterView = footView;

}






```