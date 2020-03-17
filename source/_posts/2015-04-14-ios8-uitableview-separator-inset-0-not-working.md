title: iOS8下UITableView的分隔线Inset为0不起作用
date: 2015-04-14 23:39:48
categories: 
- 技海拾贝
- iOS
tags:
- iOS
- UITableView
- Inset
---

在使用UITableView的时候，两个Cell之间的分隔线，左边没有到头，而是空了一小段，从网上找到了解决办法却不生效，最后从StackOverflow上找到了可用的解决办法。

<!-- more -->

大家普遍提供的解决办法是：

    if ([self.tableView respondsToSelector:@selector(setSeparatorInset:)]) {
        [self.tableView setSeparatorInset:UIEdgeInsetsZero];
    }

但是在iOS8上却不生效，当然，iOS8之前的版本是否生效我也不确认，因为没有试过。

在网上查了一下解决办法，最后在StackOverflow上找到了可用的解决办法：

    - (void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell forRowAtIndexPath:(NSIndexPath *)indexPath {
        if ([cell respondsToSelector:@selector(setSeparatorInset:)]) {
            [cell setSeparatorInset:UIEdgeInsetsZero];
        }
        
        if ([cell respondsToSelector:@selector(setPreservesSuperviewLayoutMargins:)]) {
            [cell setPreservesSuperviewLayoutMargins:NO];
        }
        
        if ([cell respondsToSelector:@selector(setLayoutMargins:)]) {
            [cell setLayoutMargins:UIEdgeInsetsZero];
        }
    }

StackOverflow上的地址是：http://stackoverflow.com/questions/25770119/ios-8-uitableview-separator-inset-0-not-working

记录一下~