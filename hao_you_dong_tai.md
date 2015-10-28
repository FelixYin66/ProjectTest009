# 好友动态


## 好友动态结构图

![好友动态](好友动态结构图.png)




####"好友动态"模块负责展示"当前用户"的好友到访"当前用户"所在城市的所有记录与好友最新的日程安排


到访动态效果图：


![到访动态](好友到访.png)



好友动态效果图:

![好友动态](好友动态.png)



```swift

- (void)viewDidLoad
{
    [super viewDidLoad];
    
    self.title = @"动态";

    NSArray *segArr = [[NSArray alloc]initWithObjects:@"到访动态",@"好友动态", nil];
    
    
    //默认状态，选中“到访动态”

    indexSeg = 0;
    
    //标记未请求好友动态
    
    tripRequest = NO;
    
    
    //创建UISegmentedControl 并添加到self.View中
    
    segmentedCon = [[UISegmentedControl alloc]initWithItems:segArr];
    segmentedCon.frame = CGRectMake(0, 7, 200, 30);
    segmentedCon.selectedSegmentIndex = indexSeg;
    
    if (ISIOS7) {
        segmentedCon.tintColor = [UIColor whiteColor];
    }
    
    segmentedCon.segmentedControlStyle = UISegmentedControlStylePlain;
    
    //设置监听选中事件监听方法
    
    [segmentedCon addTarget:self action:@selector(segMentEvent:) forControlEvents:UIControlEventValueChanged];
    
    //设置navigationItem的titleView为 segmentedCon

    self.navigationItem.titleView = segmentedCon;
    
    [segmentedCon release];
    
    [segArr release];


    //添加一个删除提醒的通知
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(getDataSource) name:DeleScheduleNoti object:nil];
    
    //监听系统通知
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(getDataSource) name:NotiComeFri object:nil];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(getDataSource) name:NotiDeleFri object:nil];
    
    
    //初始化内容视图

    myTable = [[UITableView alloc]initWithFrame:CGRectMake(0, 0, UIWidth, UIHeight-UI_NavY-UI_Bar) style:UITableViewStyleGrouped];
    
    myTable.delegate = self;
    myTable.dataSource = self;
    
    myTable.backgroundColor = [UIColor clearColor];
    myTable.separatorColor = [UIColor clearColor];
    myTable.separatorStyle = UITableViewCellSeparatorStyleNone;
    myTable.tableFooterView = [[[UIView alloc] init] autorelease];
    if (ISIOS7) {
        myTable.sectionIndexBackgroundColor = [UIColor clearColor];
        myTable.sectionIndexTrackingBackgroundColor = [UIColor clearColor];
        self.edgesForExtendedLayout = UIRectEdgeNone;
    }

    [self.view addSubview:myTable];
    
    
    //初始化数据数组<到访动态数组，好友动态数组>


    scheduleComeArr = [[NSMutableArray alloc]init];
    scheduleDateArr = [[NSMutableArray alloc]init];
    
    
    //添加刷新控件

    if (_refreshHeaderView == nil) {

		EGORefreshTableHeaderView *view = [[EGORefreshTableHeaderView alloc] initWithFrame:CGRectMake(0.0f, -200.0f, myTable.frame.size.width, 200.0f)];
		view.delegate = self;
        view.backgroundColor = [UIColor clearColor];
		[myTable addSubview:view];
		_refreshHeaderView = view;
		[view release];

	}

    
    //加载数据
    [self locationMyCityData];
    
    //加载网络数据，并缓存到本地
    [self getMyCityData];
    
    
    //  给UITableView添加左右滑动手势
    
    UISwipeGestureRecognizer *swipeGesture=[[UISwipeGestureRecognizer alloc]initWithTarget:self action:@selector(rightNavMenuEvent)];
    swipeGesture.direction=UISwipeGestureRecognizerDirectionLeft;
    [myTable addGestureRecognizer:swipeGesture];
    [swipeGesture release];
    
    UISwipeGestureRecognizer *swipeGesture2=[[UISwipeGestureRecognizer alloc]initWithTarget:self action:@selector(leftNavMenuEvent)];
    swipeGesture2.direction=UISwipeGestureRecognizerDirectionRight;
    [myTable addGestureRecognizer:swipeGesture2];
    [swipeGesture2 release];
    
}




///  把数据转为model
///
///  @param getScheduleArr 服务器或者本地数据数组

-(void)readLocationData:(NSArray *)getScheduleArr{
    if (getScheduleArr.count <=0 ) {
        
        if (!emptyImg) {
            emptyImg = [[[UIImageView alloc]initWithFrame:CGRectMake(0, 110, UIWidth, 125)] autorelease];
            
            //当没有数据时，向UITableView添加一个提示图片
            [myTable addSubview:emptyImg];

        } else {
            emptyImg.hidden = NO;
        }
        
        if (indexSeg == 0) {
            emptyImg.image = [UIImage imageNamed:@"空状态－暂无到访动态"];
        } else {
            //日程动态
            emptyImg.image = [UIImage imageNamed:@"空状态－暂无好友动态"];
        }
        
        //UITableView加载数据
        
        [myTable reloadData];

        return;
    } else {
        
        emptyImg.hidden = YES;
    }
    
    //清空之前所有的数据

    [scheduleComeArr removeAllObjects];

    if (indexSeg == 0) {
        
        //到访动态

        for (int i=0; i<getScheduleArr.count; i++) {
            
            NSDictionary *tempDict = [getScheduleArr objectAtIndex:i];
            ScheduleModel *scheduleModel = [[[ScheduleModel alloc]init] autorelease];
            scheduleModel.user_id = [tempDict objectForKey:@"friend_id"];
            if ([[tempDict objectForKey:@"remark"] isEqualToString:@""]||[tempDict objectForKey:@"remark"] == nil) {
                if ([[tempDict objectForKey:@"name"] isEqualToString:@""]||[tempDict objectForKey:@"name"]==nil) {
                    scheduleModel.user_name = [tempDict objectForKey:@"tel"];
                } else {
                    scheduleModel.user_name = [tempDict objectForKey:@"name"];
                }
            } else {
                scheduleModel.user_name = [tempDict objectForKey:@"remark"];
            }
            scheduleModel.user_phoneNum = [tempDict objectForKey:@"tel"];
            scheduleModel.user_headImgUrl = [tempDict objectForKey:@"avatar_url"];
            scheduleModel.schedule_ID = [tempDict objectForKey:@"schedule_id"];
            scheduleModel.schedule_city = [tempDict objectForKey:@"address"];
            scheduleModel.schedule_days = [[tempDict objectForKey:@"date_length"] intValue];
            [scheduleComeArr addObject:scheduleModel];
        }

    } else {
        
        //好友动态

        NSMutableArray *tempScheduleArr = [[[NSMutableArray alloc]init] autorelease];

        for (int j=0; j<getScheduleArr.count; j++) {

            NSDictionary *tempDict = [getScheduleArr objectAtIndex:j];
            
            //好友动态里多了一个“日程开始时间”，后面是用这个进行排序
            
            NSDate *schDate = [timeToString stringToDate:[tempDict objectForKey:@"date_start"]];

            ScheduleModel *scheduleModel = [[[ScheduleModel alloc]init] autorelease];
            scheduleModel.user_id = [tempDict objectForKey:@"uid"];
            
            if ([[tempDict objectForKey:@"remark"] isEqualToString:@""]||[tempDict objectForKey:@"remark"] == nil) {
                
                if ([[tempDict objectForKey:@"name"] isEqualToString:@""]||[tempDict objectForKey:@"name"]==nil) {
                    scheduleModel.user_name = [tempDict objectForKey:@"tel"];
                } else {
                    scheduleModel.user_name = [tempDict objectForKey:@"name"];
                }
                
            } else {
                scheduleModel.user_name = [tempDict objectForKey:@"remark"];
            }
            
            scheduleModel.user_phoneNum = [tempDict objectForKey:@"tel"];
            scheduleModel.user_headImgUrl = [tempDict objectForKey:@"avatar_url"];
            scheduleModel.schedule_ID = [tempDict objectForKey:@"schedule_id"];
            scheduleModel.schedule_city = [tempDict objectForKey:@"address"];
            scheduleModel.schedule_days = [[tempDict objectForKey:@"date_length"] intValue];
            scheduleModel.schedule_date_year = [timeToString returnYear:schDate];
            scheduleModel.schedule_date_month = [timeToString returnMonth:schDate];
            scheduleModel.schedule_date_day = [timeToString returnDay:schDate];

            [tempScheduleArr addObject:scheduleModel];
        }
        
        //”好友动态“按照时间排序....
        [self schDayArr:tempScheduleArr];

    }
    
    //刷新UITableView
    
    [myTable reloadData];

}




```






