# 生物动力法


## 生物动力发架构图

![生物动力法](生物动力法.png)


"生物动力法"根据生物动力年历，把时间分为"花日"，"果日"，"根日"，"叶日","空日"五种情况，根据每天所对时间类型，看看今天是否适宜喝葡萄酒或者不宜饮酒，适合喝什么酒.


## 涉及文件名说明：

| ```CountryViewController``` | 此代表的是展示所有的国家视图控制器 |
| -- | -- |
| ```CountryTableViewCell``` | 此代表的是单个国家记录视图，是UITableView中的一个子控件 |
| ```CityModel``` | 此代表的是数据实体对象<城市实体> |






选择你想了解国家生物动力法：

![选择国家](国家.png)



```swift

//将读取出来的数据转换成model实体

-(void)dataToModel:(NSArray *)dataArr {
    
    
    //循环遍历转换model
    
    // country 表示所在国家   city 表示所在国家城市名  timeZone表示时差
    
    for (int i=0; i<dataArr.count; i++) {
        CityModel *_cityModel = [[CityModel alloc]init];
        _cityModel.CC_Name = [NSString stringWithFormat:@"%@%@",[dataArr[i] objectForKey:@"country"],[dataArr[i] objectForKey:@"city"]];
        _cityModel.country_Name = [dataArr[i] objectForKey:@"country"];
        _cityModel.city_Name = [dataArr[i] objectForKey:@"city"];
        _cityModel.location_Name = [dataArr[i] objectForKey:@"timeZone"];
        [cityModelArr addObject:_cityModel];
    }
    
    
    //将解析后的数组进行排序
    
    cityModelArr = [SortObject sortArrayWithPinYin:cityModelArr];
    [sortedArrForArrays addObjectsFromArray:[self getChineseStringArr:cityModelArr]];
}




-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {

    static NSString *cellIdentifier = @"cell";
    CountryTableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:cellIdentifier];
    
    //当没有cell时，创建一个cell
    
    if (cell == nil) {
        cell = [[CountryTableViewCell alloc]initWithStyle:UITableViewCellStyleValue1 reuseIdentifier:cellIdentifier];
        [cell setSelectionStyle:UITableViewCellSelectionStyleNone];
    }
    
    
    //但是第一组时，展示此时用户所在的国家
    
    if (indexPath.section == 0) {
        cell.engNameLab.textColor = ProColorWhite;
        cell.locationLab.textColor = ProColorWhite;
        cell.backImg.image = [UIImage imageNamed:@"国家选择-定位"];
        cell.cityModel = locationCity;
    } else {
        
        
        //其他组的情况
        
        cell.engNameLab.textColor = ProColorGray;
        cell.locationLab.textColor = ProColorGray;
        
        //设置当前索引下的cityModel
        
        CityModel *tempModel = [[sortedArrForArrays objectAtIndex:indexPath.section-1] objectAtIndex:indexPath.row];
    

        cell.cityModel = tempModel;
        
        //按照奇偶，设置不同的背景图片
        
        if (indexPath.row%2==0) {
            cell.backImg.image = [UIImage imageNamed:@"国家选择-蓝"];
        } else {
            cell.backImg.image = [UIImage imageNamed:@"国家选择-粉"];
        }
        
    }
    
    return cell;
}




```
