---
layout: post
title:  "解决输入的textField和textView被键盘遮挡的问题"
date:   2015-08-07 15:18:05
categories: IOS
excerpt: ScrollView GridView ListView
---

## 问题

在IOS编程中，如果一样页面的输入框比较低，在键盘弹出来的时候，总是遮挡住输入框。本文是处理这种情况的一个解决方案。

## 解决方法

通告键这样的处理方法编写一个工具类，主要参考了官方的方法。

* 下面是头文件

{%  highlight c %}

// KeyboardHandler.h

#import <Foundation/Foundation.h>

typedef enum DragDirectionForEnd
{
    DRAG_DIRECTION_NORMAL, // 上下滑皆可终止
    DRAG_DIRECTION_TOP,    // 上滑终止
    DRAG_DIRECTION_BOTTOM  // 下滑终止
} DragDirections;

@protocol KeyboardHandlerDelegate <NSObject>

- (UIView*) getPresentedView;

@end

@interface KeyboardHandler : NSObject

@property (nonatomic, weak) UIScrollView* scrollView;
@property (nonatomic, assign) id<KeyboardHandlerDelegate> delegate;
@property (nonatomic) DragDirections dragdirection;

- (instancetype)initWithScrollView:(UIScrollView*)scrollView;
- (void)start;
- (void)stop;

@end

{%  endhighlight %}

* 下面是实现文件

{%  highlight c %}

#import "KeyboardHandler.h"

@interface KeyboardHandler()<UIScrollViewDelegate>
{
    CGFloat kbHeight;
    CGFloat lastContentOffset;
}

@end

@implementation KeyboardHandler
@synthesize scrollView;
@synthesize dragdirection;

- (instancetype) initWithScrollView:(UIScrollView *)scrollVie
{
    self = [super init];
    if (self) {
        scrollView = scrollVie;
        scrollView.delegate = self;
        dragdirection = DRAG_DIRECTION_NORMAL;
    }
    return self;
}
- (void)start
{
    [self registerForKeyboardNotifications];
}

- (void)stop
{
    [[NSNotificationCenter defaultCenter] removeObserver:self ];
}

- (void)dealloc
{
    [[NSNotificationCenter defaultCenter] removeObserver:self ];
}

# pragma - 键盘遮挡处理

- (void)registerForKeyboardNotifications
{
    [[NSNotificationCenter defaultCenter] addObserver:self
                                             selector:@selector(keyboardWasShown:)
                                                 name:UIKeyboardWillChangeFrameNotification object:nil];
    
    [[NSNotificationCenter defaultCenter] addObserver:self
                                             selector:@selector(keyboardWillBeHidden:)
                                                 name:UIKeyboardWillHideNotification object:nil];
    
}

- (void) keyboardWasShown:(NSNotification*)aNotification
{
    UIView* presented = nil;
    if (_delegate) {
        presented = [_delegate getPresentedView];
        if (!presented) {
            return;
        }
    }else {
        return;
    }
    NSDictionary* info = [aNotification userInfo];
    CGSize kbSize = [[info objectForKey:UIKeyboardFrameEndUserInfoKey] CGRectValue].size;
    if (kbSize.height == kbHeight) {
        return ;
    } else if (kbSize.height < kbHeight){
        
        // 处理键盘高度变小时的情况
        CGFloat offSet = kbSize.height - kbHeight;
        CGPoint contentOffset = scrollView.contentOffset;
        contentOffset.y += offSet;
        [scrollView setContentOffset:contentOffset animated:YES];
    }
    kbHeight = kbSize.height;
    UIEdgeInsets contentInsets = UIEdgeInsetsMake(0.0, 0.0, kbSize.height, 0.0);
    self.scrollView.contentInset = contentInsets;
    self.scrollView.scrollIndicatorInsets = contentInsets;
    CGRect screenRect = [[UIScreen mainScreen] bounds];
    screenRect.size.height -= kbSize.height;
    
    // 将输入框的左下角的点移动到可视
    CGPoint presentedAbsolutePosition = [presented.superview convertPoint:presented.frame.origin toView:nil];
    presentedAbsolutePosition.y += presented.frame.size.height;

    if (!CGRectContainsPoint(screenRect, presentedAbsolutePosition)) {
       [self.scrollView scrollRectToVisible:presented.frame animated:YES];
    }
}

- (void) keyboardWillBeHidden:(NSNotification*)aNotification
{
    UIEdgeInsets contentInsets = UIEdgeInsetsMake(0.0, 0.0, 0.0, 0.0);;
    scrollView.contentInset = contentInsets;
    scrollView.scrollIndicatorInsets = contentInsets;
    kbHeight = 0;
}

#pragma scrollview

// 手势滑动开始
- (void)scrollViewWillBeginDragging:(UIScrollView *)scroll
{
    if (dragdirection == DRAG_DIRECTION_NORMAL) {
        UIView* view = [_delegate getPresentedView];
        if ([view respondsToSelector:@selector(endEditing:)]){
            [[_delegate getPresentedView] endEditing:YES];
        }
    } else {
        lastContentOffset = scroll.contentOffset.y;
    }
}
// 手势滑动结束
- (void)scrollViewDidEndDragging:(UIScrollView *)ascrollView willDecelerate:(BOOL)decelerate
{
    if (dragdirection != DRAG_DIRECTION_NORMAL) {
        if (lastContentOffset > ascrollView.contentOffset.y && dragdirection == DRAG_DIRECTION_BOTTOM) {
            UIView* view = [_delegate getPresentedView];
            if ([view respondsToSelector:@selector(endEditing:)]){
                [[_delegate getPresentedView] endEditing:YES];
            }
        } else if (lastContentOffset < ascrollView.contentOffset.y && dragdirection == DRAG_DIRECTION_TOP){
            UIView* view = [_delegate getPresentedView];
            if ([view respondsToSelector:@selector(endEditing:)]){
                [[_delegate getPresentedView] endEditing:YES];
            }
        }
    }

    
}

@end

{%  endhighlight %}

* 使用方法

{%  highlight c %}

@interface TestController () <KeyboardHandlerDelegate>
{
    UIView* edittingView;
    KeyboardHandler* keyboardHandler;
}

@property (strong, nonatomic) IBOutlet UITextView *yourInputView;

@end

@implementation TestController

- (void)viewDidLoad
{
    [super viewDidLoad];
    keyboardHandler = [[KeyboardHandler alloc] initWithScrollView:self.scrollView];
    keyboardHandler.delegate = self;
    yourInputView.delegate = self;
}

- (void)viewWillAppear:(BOOL)animated
{
    [keyboardHandler start];
}

- (void)viewDidDisappear:(BOOL)animated
{
    [keyboardHandler stop];
}

#pragma mark - KeyboardHandlerDelegate

- (UIView*)getPresentedView
{
    return edittingView;
}

- (BOOL)textViewShouldBeginEditing:(UITextView *)textView
{
    edittingView = textView;
    return true;
}

- (BOOL)textFieldShouldBeginEditing:(UITextField *)textField
{
    edittingView = textField;
    return YES;
}

{%  endhighlight %}