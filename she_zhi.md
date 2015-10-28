# 设置

![设置模块](设置模块1.png)


"设置"可给当前用户设置“个性图像”，还可以“修改密码”，“App评分”，“意见反馈”，“退出登录”，此模块功能比较零散


```swift

-(void)ccActionSheet:(CCActiotSheet *)actionSheet clickedButtonAtIndex:(NSInteger)buttonIndex {
    if (actionSheet.tag == 888) {

        if (buttonIndex == 1) {
            [self shareWx:WXSceneTimeline];
        } else if (buttonIndex == 2) {
            [self shareWx:WXSceneSession];
        } else if (buttonIndex == 3) {

            [SVProgressHUD showWithStatus:@"正在分享"];
            [ShareClass mainRoot].share_type = SINA;
            
            [[NSUserDefaults standardUserDefaults]setObject:@"share" forKey:@"isShare"];
            [[NSUserDefaults standardUserDefaults]synchronize];

            //判断是否授权过期了。过期了，就去授权。
            if ([[NSUserDefaults standardUserDefaults]objectForKey:@"user_sinaToken"] == nil) {

                sinaBind = NO;
                [self sinaSSO];
                [myTable reloadData];
                return;
            } else {
                NSTimeInterval nowTime = [[NSDate date] timeIntervalSince1970];
                double authTime = [[[NSUserDefaults standardUserDefaults]objectForKey:@"authTime"] doubleValue]; //上次授权时间
                double tempTime = [[[NSUserDefaults standardUserDefaults] objectForKey:@"expires_in"] doubleValue]; //授权有效时间段

                NSLog(@"nowtime %f passtime %f passtime %f",nowTime,authTime,tempTime);
                if (authTime+tempTime<=nowTime) {
                    sinaBind = NO;
                    [self sinaSSO];
                    [myTable reloadData];
                    return;
                }
            }

            //下面是没有过期，直接分享。
            
            NSString *strText = [NSString stringWithFormat:@"#碰碰日程# 上一次见面是什么时候的事了？你在另一个城市过得还好么？我正在使用“碰碰日程”，共享出行日程，相聚并没有那么难！拉近我们的距离，你也下载一个吧！%@", QianXianUrl];
//            NSData *data = [NSData dataWithContentsOfFile:[[NSBundle mainBundle] pathForResource:@"111" ofType:@"png"]];
            NSData *data = UIImageJPEGRepresentation([UIImage  imageNamed:@"碰碰日程分享页"], 1.0);
            
            [CCInterface shareSinaWeiboWithAppkey:kAppKey access_token:[[NSUserDefaults standardUserDefaults] objectForKey:@"user_sinaToken"] text:strText image:data backBlock:^(int status, NSDictionary *dictResult) {
                if (status == 200) {

                    if ([dictResult objectForKey:@"error_code"] == nil) {

                        if ([dictResult objectForKey:@"id"]) {

                            [self showWait:@"分享成功"];
                            [self performSelector:@selector(hideWait) withObject:nil afterDelay:1.0];

                        }
                    }else{

                        //重复分享
                        if ([[dictResult objectForKey:@"error_code"] intValue] == 20111) {

                            [self showWait:@"重复分享 :("];
                            [self performSelector:@selector(hideWait) withObject:nil afterDelay:1.0];

                        }else if([[dictResult objectForKey:@"error_code"] intValue] == 21315 || [[dictResult objectForKey:@"error_code"] intValue] == 21316 ||[[dictResult objectForKey:@"error_code"] intValue] == 21317 ) {
                            sinaBind = NO;
                            [self sinaSSO];
                            [myTable reloadData];


                        }else{
                            //失败
                            [self showWait:@"分享失败 :("];
                            [self performSelector:@selector(hideWait) withObject:nil afterDelay:1.0];

                        }

                    }


                }else{
                    //失败
                    [self showWait:@"分享失败 :("];
                    [self performSelector:@selector(hideWait) withObject:nil afterDelay:1.0];
                    
                }
            }];
            

            
          
        }
        
    // actionSheet.tag = 1888 此选项为上传图片操作
    } else if (actionSheet.tag == 1888) {

        NSUInteger sourceType = 0;
        // 判断是否支持相机
        if([UIImagePickerController isSourceTypeAvailable:UIImagePickerControllerSourceTypeCamera]) {

            switch (buttonIndex) {
                case 1:
                    //访问相机
                    sourceType = UIImagePickerControllerSourceTypeCamera;
                    break;
                case 2:
                    //访问相册
                    sourceType = UIImagePickerControllerSourceTypePhotoLibrary;
                    break;
                case 0:
                    //不上传取消
                    return;
            }
            
        // 手机相册不可用
            
        }else {
            
            if (buttonIndex == 1 || buttonIndex == 2) {
                sourceType = UIImagePickerControllerSourceTypeSavedPhotosAlbum;
            } else {
                return;
            }
            
        }
        
        // 跳转到相机或相册页面
        
        // 初始化UIImagePickerController 访问手机相册
        
        UIImagePickerController *imagePickerController = [[UIImagePickerController alloc] init];

        imagePickerController.delegate = self;

        imagePickerController.allowsEditing = YES;
        
        //设置访问相机的类型

        imagePickerController.sourceType = sourceType;
        
        //modal出imagePickerController
        
        [self presentViewController:imagePickerController animated:YES completion:^{}];
        
        [imagePickerController release];
        
        //前期员工添加的...及其消耗性能

//        timer = [NSTimer timerWithTimeInterval:0.1 target:self selector:@selector(test) userInfo:self repeats:YES];
//        [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
        
        
    // actionSheet.tag = 2888 代表是退出当前用户登录
        
    } else if (actionSheet.tag == 2888) {
        
        if (buttonIndex == 1) {
            
            
            //退出登录时，进入到登录界面

            AppDelegate *dele = (AppDelegate *)[[UIApplication sharedApplication]delegate];
            [dele userLogin];
            
            //当用户退出登录，发送UserIDChangeNoti通知
            [[NSNotificationCenter defaultCenter] postNotificationName:UserIDChangeNoti object:nil];
        }

    }

}


```

设置个性化图像：


![设置图像](设置图像.png)


```swift
//当拍照或者选中相册中图片结束后调用此方法

-(void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary *)info {

    //获取从手机得到的图片，并设置其半径大小为59的UIImage
    
    UIImage *uploadImg = [UIImage createRoundedRectImage:[info objectForKey:UIImagePickerControllerEditedImage] size:CGSizeMake(59*2, 59*2) radius:59];
    
    
    //UIImageJPEGRepresentation(uploadImg, 1.0) 将UIImage转换成二进制文件，并以原比例保存图片

    [CCInterface requestUploadHeadImg:UIImageJPEGRepresentation(uploadImg, 1.0) backBlock:^(int status, NSDictionary *dictResult) {
        
        NSLog(@"上传图片结果 %@",dictResult);
        
        //退出imagePickerView控制器
        
        [picker dismissViewControllerAnimated:YES completion:nil];

        if (status == 200) {
            if ([[dictResult objectForKey:@"status"] isEqualToString:statusSuccess]) {
                [SVProgressHUD showSuccessWithStatus:[dictResult objectForKey:@"msg"]];
                
                //保存个性图像的imgURL,此路径为服务器中UIImage的路径

                userModel.user_headImgUrl = [dictResult objectForKey:@"avatar_url"];
                
                //设置headerView中UIImage中图像

                ((UIImageView *)[self.view viewWithTag:199]).image = uploadImg;

            } else {
                [SVProgressHUD showErrorWithStatus:[dictResult objectForKey:@"msg"]];
            }
        }
    }];

}



```


修改密码：

![修改密码](修改密码.png)


```swift


///  点击“确认”按钮触发事件


-(void)loginEvent {
    
    
    //旧密码

    NSString *pwStr1 = ((CSTextField *)[self.view viewWithTag:100]).text;
    
    //新密码
    
    NSString *pwStr2 = ((CSTextField *)[self.view viewWithTag:101]).text;
    
    //确认密码
    
    NSString *pwStr3 = ((CSTextField *)[self.view viewWithTag:102]).text;
    
    
    //除去三个输入框中的空格

    pwStr1 = [pwStr1 stringByReplacingOccurrencesOfString:@" " withString:@""];
    pwStr2 = [pwStr2 stringByReplacingOccurrencesOfString:@" " withString:@""];
    pwStr3 = [pwStr3 stringByReplacingOccurrencesOfString:@" " withString:@""];

    NSString *message = nil;
    
    
    //输入框信息验证

    if (pwStr1.length == 0) {
        message = @"当前密码不能为空";
        [SVProgressHUD showErrorWithStatus:message];
    } else if (pwStr2.length == 0) {
        message = @"新密码不能为空";
        [SVProgressHUD showErrorWithStatus:message];
    } else if (pwStr3.length == 0) {
        message = @"确认新密码不能为空";
        [SVProgressHUD showErrorWithStatus:message];
    } else if (pwStr2.length<6 || pwStr3.length>20) {//|| pwStr2.length>11 || pwStr3.length>11
        message = @"请输入6-20位新密码";
        [SVProgressHUD showErrorWithStatus:message];
    }else if (![pwStr2 isEqualToString:pwStr3]) {
        message = @"两次输入新密码不一致";
        [SVProgressHUD showErrorWithStatus:message];
    }else if ([pwStr1 isEqualToString:pwStr2]){
        message = @"当前密码与新密码重复";
        [SVProgressHUD showErrorWithStatus:message];
    } else {
        
        //格式正确，发送”修改密码“请求

        [CCInterface requestUpdatePassWord:pwStr1 newPW:pwStr2 backBlock:^(int status, NSDictionary *dictResult) {

            NSLog(@"dict is %@",dictResult);
            
            if (status == 200) {
                if ([[dictResult objectForKey:@"status"] isEqualToString:statusSuccess]) {
                    
                    //修改成功提醒
                    
                    [SVProgressHUD showSuccessWithStatus:[dictResult objectForKey:@"msg"]];
                    [self.navigationController popViewControllerAnimated:YES];
                } else {
                    
                    //错误提醒
                    
                    [SVProgressHUD showErrorWithStatus:[dictResult objectForKey:@"msg"]];
                }
            }
        }];
        
    }
}





```


意见反馈：

![意见反馈](意见反馈.png)

```swift

///  点击发送“意见反馈”按钮

-(void)sendEvent {
    
    NSString *opinionStr = ((UITextView *)[self.view viewWithTag:1001]).text;
    opinionStr = [opinionStr stringByReplacingOccurrencesOfString:@" " withString:@""];
    
    
    //输入框字符数验证

    if (opinionStr.length>300) {
        
        UIAlertView *myAlert = [[UIAlertView alloc]initWithTitle:@"字数超限" message:[NSString stringWithFormat:@"请控制在300字以内，当前为%ld个字符", (unsigned long)opinionStr.length] delegate:nil cancelButtonTitle:@"取消" otherButtonTitles:@"确定", nil];
        [myAlert show];
        [myAlert release];
        return;
    } else if (opinionStr.length==0) {
        [SVProgressHUD showErrorWithStatus:@"请输入反馈内容"];
        return;
    }
    
    //当反馈内容小于300字符时，发送网络请求
    
    [CCInterface requestFeekback:((UITextView *)[self.view viewWithTag:1001]).text contact:((UITextView *)[self.view viewWithTag:1000]).text backBlock:^(int status, NSDictionary *dictResult) {
        NSLog(@"反馈 %@",dictResult);

        if (status == 200) {
            if ([[dictResult objectForKey:@"status"] isEqualToString:statusSuccess]) {
                [SVProgressHUD showSuccessWithStatus:[dictResult objectForKey:@"msg"]];
                [self.navigationController popViewControllerAnimated:YES];
            } else {
                [SVProgressHUD showErrorWithStatus:[dictResult objectForKey:@"msg"]];
            }
        }
    }];

}



```


退出登录:

![退出登录](退出登录.png)

```swift

  if (actionSheet.tag == 2888) {
        
        if (buttonIndex == 1) {
            
            
            //退出登录时，进入到登录界面

            AppDelegate *dele = (AppDelegate *)[[UIApplication sharedApplication]delegate];
            [dele userLogin];
            
            //当用户退出登录，发送UserIDChangeNoti通知
            [[NSNotificationCenter defaultCenter] postNotificationName:UserIDChangeNoti object:nil];
        }

    }



```


