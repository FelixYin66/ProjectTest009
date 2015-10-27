# 好友

此模块负责展示已添加好友，并查看是否有新的好友请求，当前用户可发送“添加好友”的请求，展示好友是按照A->Z排序分组展示，当好友特别多的时候可以使用搜索功能搜索你的好友.



![好友](好友模块.png)





好友列表效果图：


![好友列表](展示好友1.png)



```swift

#pragma mark 把好友转为model

-(void)readLocationData:(NSArray *)myScheduleArr{
    
    
    //清空之前保存下来的”首字母“数组
    
    [sectionHeadsKeys removeAllObjects];
    [sectionHeadsKeys addObject:@""];
    
    //清空之前保存下来的好友数组
    
    [muArrFriends removeAllObjects];
    
    //    清空之前保存下来的排序后的数组
    
     [sortedArrForArrays removeAllObjects];
    
    //清空当前用户的user_friendsArr值
    
    [userModel.user_friendsArr removeAllObjects];
    
    NSLog(@"myarr is %ld",(unsigned long)myScheduleArr.count);
    
    
    
    //判断是否有好友
    
    if (myScheduleArr.count <=0 ) {
        
        if (!emptyImg) {
            emptyImg = [[[UIImageView alloc]initWithFrame:CGRectMake(0, 110, UIWidth, 145)] autorelease];
            emptyImg.image = [UIImage imageNamed:@"空状态－暂无好友"];
            emptyImg.hidden = YES;
            [myTableView addSubview:emptyImg];

            emptyImg.userInteractionEnabled = YES;
            UITapGestureRecognizer * tap = [UITapGestureRecognizer new];
            [tap addTarget:self action:@selector(handleTap)];
            [emptyImg addGestureRecognizer:tap];

        } else {
            
            emptyImg.hidden = NO;
            
        }
        
        
        //刷新myTableView的数据

        [myTableView reloadData];

        return;
        
    } else {
        
        //当有好友时，emptyImg为隐藏状态
        
        emptyImg.hidden = YES;
        
    }
    
    

    //循环遍历数据，将字典转换成UserModel模型
    
    for (int m=0; m<[myScheduleArr count]; m++) {
        NSDictionary *tempDict = [myScheduleArr objectAtIndex:m];
        UserModel *friModel = [[[UserModel alloc]init] autorelease];
        friModel.user_id = [tempDict objectForKey:@"friend_id"];
        if ([[tempDict objectForKey:@"remark"] isEqualToString:@""]||[tempDict objectForKey:@"remark"] == nil) {
            if ([[tempDict objectForKey:@"name"] isEqualToString:@""]||[tempDict objectForKey:@"name"]==nil) {
                friModel.user_name = [tempDict objectForKey:@"tel"];
            } else {
                friModel.user_name = [tempDict objectForKey:@"name"];
            }
        } else {
            friModel.user_name = [tempDict objectForKey:@"remark"];
        }
        friModel.user_remark = [tempDict objectForKey:@"remark"];
        friModel.user_nickName = [tempDict objectForKey:@"name"];
        friModel.user_phoneNum = [tempDict objectForKey:@"tel"];
        friModel.user_headImgUrl = [tempDict objectForKey:@"avatar_url"];
        friModel.user_homeCity = [tempDict objectForKey:@"address"];
        if ([[tempDict objectForKey:@"new_message"] intValue]>0) {
            friModel.user_isChanged = YES;
        }else {
            friModel.user_isChanged = NO;
        }
        friModel.user_relation = 4;
        [userModel.user_friendsArr addObject:friModel];
        
    }

    [muArrFriends addObjectsFromArray:userModel.user_friendsArr];
    muArrFriends = [SortObject sortArrayWithPinYin:muArrFriends];
    
    //将数组按照首字母A-Z进行排序
    
    [sortedArrForArrays addObjectsFromArray:[self getChineseStringArr:muArrFriends]];

    
    //  有数据时，tableView加载数据

    [myTableView reloadData];


}


```



添加好友效果图：


![添加好友1](添加好友1.png)

```swift




```


![添加好友2](添加好友2.png)