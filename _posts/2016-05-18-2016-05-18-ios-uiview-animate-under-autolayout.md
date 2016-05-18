---
layout: blog
categories: blog
published: false
title: "2016-05-18-iOS-UIView-animate-under-autolayout"
---
# Scene

Move the view to right handside, and out of the screen right boundary, until the whole view is not visible, and stop.


## What is the problem when changing the frame

If we use the following code to animate the view, then the outcome is not as expected. The autolayout system will set the view constraints to the storyborad settings after the animation completed, so you will lost the animation result, and the animation will be weird.

    CGFloat barHeight = 0;
    CGFloat screenWidth = 0;
    if (SYSTEM_VERSION_LESS_THAN(@"8.0")) {
        barHeight = [[UIScreen mainScreen] bounds].size.width;
        screenWidth = [[UIScreen mainScreen] bounds].size.height;
    } else {
        barHeight = [[UIScreen mainScreen] bounds].size.height;
        screenWidth = [[UIScreen mainScreen] bounds].size.width;
    }

    CGRect newRect = CGRectMake(0, 0, kRBNaviBarWidth, barHeight);
    newRect.origin.x = screenWidth - kRBNaviBarWidth;
    newRect.origin.y = 0;

    [UIView animateWithDuration:kRBNaviShowHideInterval animations:^{
        self.frame = newRect;
        self.alpha = kRBNaviBarAlpha;
    } completion:^(BOOL finished) {
    }];


## Autolayout constraints solution

We need to find out the trailing constraint programitically, and replace the constraint constant with the view width.

    [self.superview.constraints enumerateObjectsUsingBlock:^(NSLayoutConstraint *constraint, NSUInteger idx, BOOL *stop) {
        if (constraint.secondItem == self && constraint.secondAttribute == NSLayoutAttributeTrailing) {
            constraint.constant = -kRBNaviBarWidth;
        }
    }];

    [UIView animateWithDuration:kRBNaviShowHideInterval animations:^{
        self.alpha = 0;
        [self layoutIfNeeded];
    }];
