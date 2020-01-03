---
layout: post
title: "时间日历控件"
date: 2016-05-31 13:00:03 +0800
comments: true
tags:
- [NSCalendar]
categories:
- [日历]
- [OC]
---

关于日历就看这一个类`NSCalendar`就可以，结合日期拆解析类`NSDateComponents`就可以搞定日历相关各种奇葩问题了

<!-- more -->

## 日历时间生成

关于日历就看这一个类`NSCalendar`就可以，结合日期拆解析类`NSDateComponents`就可以搞定日历相关各种奇葩问题了

```
/**
 *  获取当前日期是周几
 *
 *  @return 输出结果是星期几的字符串。
 */
-(NSString *)weekday
{
    NSDateComponents *comps = [calendar components:NSCalendarUnitWeekday fromDate:self];
    
    NSArray *weekdays = [NSArray arrayWithObjects: [NSNull null], @"0", @"1", @"2", @"3", @"4", @"5", @"6", nil];
    
    return [weekdays objectAtIndex:comps.weekday];
}

-(NSString *)weekdayString
{
    NSString *week = [self weekday];
    
    NSDictionary *map = @{@"0" : @"周日",
                          @"1" : @"周一",
                          @"2" : @"周二",
                          @"3" : @"周三",
                          @"4" : @"周四",
                          @"5" : @"周五",
                          @"6" : @"周六",
                          };
    
    return map[week];
}

```

```

/**
 *  获取当前日期所在月的第一天是周几
 *
 *  @return 输出结果是星期几的字符串。
 */
-(NSString *)firstWeekdayInCurrentMonth
{
    double interval = 0;
    NSDate *beginDate = nil;
    NSDate *endDate = nil;
    NSCalendar *calendar = [[NSCalendar alloc] initWithCalendarIdentifier:NSCalendarIdentifierGregorian];
    calendar.timeZone = [NSTimeZone timeZoneWithName:@"Asia/Shanghai"];;
    
    [calendar setFirstWeekday:2];//设定周一为周首日
    BOOL ok = [calendar rangeOfUnit:NSCalendarUnitMonth startDate:&beginDate interval:&interval forDate:self];
    //分别修改为 NSDayCalendarUnit NSWeekCalendarUnit NSYearCalendarUnit
    if (ok) {
        endDate = [beginDate dateByAddingTimeInterval:interval-1];
    }else {
        return @"";
    }
    
    return [beginDate weekday];
}

```

```

/**
 *  获取当前日期所在月有多少天
 *
 *  @return 天数
 */
-(NSInteger)daysInCurrentMonth
{
    NSRange range =[calendar rangeOfUnit:NSCalendarUnitDay inUnit:NSCalendarUnitMonth forDate:self];
    
    return range.length;
}

```

```

/**
 *  获取当前日期所在月的下offset个月
 *
 *  @return 获得时间
 */
-(NSDate *)dateOffsetMonth:(NSInteger)offset
{
    NSDateComponents *comps = [[NSDateComponents alloc] init];
    comps.month = offset;
    
    return [calendar dateByAddingComponents:comps toDate:self options:0];
}

```

```

/**
 *  获取当前日期的下offset个日期
 *
 *  @return 获得时间
 */
-(NSDate *)dateOffsetDay:(NSInteger)offset
{
    NSDateComponents *comps = [[NSDateComponents alloc] init];
    comps.day = offset;
    
    return [calendar dateByAddingComponents:comps toDate:self options:0];
}

```

```

+(NSDate *)dateFromSelectIndex:(NSIndexPath *)indexPath
{
    NSDate *date = [[NSDate date] dateOffsetMonth:indexPath.section];
    
    NSInteger day = indexPath.row - [date firstWeekdayInCurrentMonth].integerValue + 1;
    
    NSDateComponents *comp = [[NSDateComponents alloc] init];
    comp.year = date.components.year;
    comp.month = date.components.month;
    comp.day = day;
    
    return [calendar dateFromComponents:comp];
}

-(NSIndexPath *)indexPath
{
    // 上面的反推 monthOffset 要注意，年末最后一天 和 下一年的一月的处理，所以通过判断若两者月相等section就等于0，不等的话，有可能是大于，有可能是小于(比如当前月2016-12，下个月2017-01)
    NSInteger row = self.components.day-1+[self firstWeekdayInCurrentMonth].integerValue;
    NSInteger monthOffset = (self.components.month == [NSDate date].components.month) ? 0 : 1;
    
    return [NSIndexPath indexPathForRow:row inSection:monthOffset];
}

```


## 日历时间UI相关

显示日历大都是UICollectionView 这个很简单，想怎么显示直接写cell就行。

```

#pragma mark collectionview delegate

- (NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView
{
    return 2;
}

- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section {
    
    NSDate *date = [[NSDate date] dateOffsetMonth:section];
    
    return [date daysInCurrentMonth] + [date firstWeekdayInCurrentMonth].integerValue;
}

- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath {
    
    DatePickerCell *cell =[collectionView dequeueReusableCellWithReuseIdentifier:datePickerCellReuse forIndexPath:indexPath];
    
    [cell updateCellIndex:indexPath select:(_selectIndex == indexPath)];
    
    return cell;
}

- (UICollectionReusableView *)collectionView:(UICollectionView *)collectionView viewForSupplementaryElementOfKind:(NSString *)kind atIndexPath:(NSIndexPath *)indexPath
{
    if ([kind isEqualToString:UICollectionElementKindSectionHeader]) {
        
        DatePickerHeaderView * headerCell = [collectionView dequeueReusableSupplementaryViewOfKind:UICollectionElementKindSectionHeader withReuseIdentifier:datePickerHeaderReuse forIndexPath:indexPath];
        
        NSDate *date = [[NSDate date] dateOffsetMonth:indexPath.section];
        
        headerCell.tipLab.text = [date ymString];
        
        return headerCell;
    }else if ([kind isEqualToString:UICollectionElementKindSectionFooter]){
        DatePickerFooterView * footerCell = [collectionView dequeueReusableSupplementaryViewOfKind:UICollectionElementKindSectionFooter withReuseIdentifier:datePickerFooterReuse forIndexPath:indexPath];
        footerCell.hidden = !(indexPath.section == 0);
        
        return footerCell;
    }
    
    return nil;
}

- (void)collectionView:(UICollectionView *)collectionView didSelectItemAtIndexPath:(NSIndexPath *)indexPath{
    
    [collectionView deselectItemAtIndexPath:indexPath animated:YES];
    
    _selectIndex = indexPath;
    
    [collectionView reloadData];
    
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, 0.4*NSEC_PER_SEC), dispatch_get_main_queue(), ^{
        
        if (selectBlock) {
            selectBlock();
        }
        
        [self.navigationController popViewControllerAnimated:YES];
    });
}

```

详细完整工程，请查看 [github](https://github.com/liu3399shuai/DatePicker)
