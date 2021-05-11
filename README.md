# Example of searching data source  

In case if you have large project that is processing and displaying data from multiple sources you can use method swizzling for searching source.   

For example, we have data source that calls NSFileManager -> displayNameAtPath. We want to find labels that display information from this data source. You can exchange (swizzle) original method with your custom method in NSFileManager and NSTextField in dispatch table. Than check string key in NSTextField and if it exists - highlight NSTextField.  

```objectivec

@implementation NSFileManager (FileManagerExtension)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        [NSFileManager exchangeDisplayNameAtPath];
    });
}

+ (void)exchangeDisplayNameAtPath {
    Class class = [self class];
    
    SEL originalSelector = @selector(displayNameAtPath:);
    SEL swizzledSelector = @selector(xxx_displayNameAtPath:);
    
    Method originalMethod = class_getInstanceMethod(class, originalSelector);
    Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);

    BOOL didAddMethod = class_addMethod(class,
                                        originalSelector,
                                        method_getImplementation(swizzledMethod),
                                        method_getTypeEncoding(swizzledMethod));
    
    if (didAddMethod) {
        class_replaceMethod(class,
                            swizzledSelector,
                            method_getImplementation(originalMethod),
                            method_getTypeEncoding(originalMethod));
    } else {
        method_exchangeImplementations(originalMethod, swizzledMethod);
    }
}

- (NSString *)xxx_displayNameAtPath:(NSString *)path {
    NSString *displayName = [self xxx_displayNameAtPath:path];
    NSString *key = [NSFileManager KeyNameReceivedFromDisplayNameAtPathMethod];
    return [NSString stringWithFormat:@"%@%@", key, displayName];
}

+ (NSString *)KeyNameReceivedFromDisplayNameAtPathMethod {
    return @"Key_NameReceivedFromDisplayNameAtPathMethod_";
}

@end

@implementation NSTextField (TextFieldExtension)

+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        [NSTextField exchangeSetStringValue];
    });
}

+ (void)exchangeSetStringValue {
    Class class = [self class];
    
    SEL originalSelector = @selector(setStringValue:);
    SEL swizzledSelector = @selector(new_setTitle:);
    
    Method originalMethod = class_getInstanceMethod(class, originalSelector);
    Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);

    BOOL didAddMethod = class_addMethod(class,
                                        originalSelector,
                                        method_getImplementation(swizzledMethod),
                                        method_getTypeEncoding(swizzledMethod));
    
    if (didAddMethod) {
        class_replaceMethod(class,
                            swizzledSelector,
                            method_getImplementation(originalMethod),
                            method_getTypeEncoding(originalMethod));
    } else {
        method_exchangeImplementations(originalMethod, swizzledMethod);
    }
}

- (void)new_setTitle:(NSString *)title {
    NSString *key = [NSFileManager KeyNameReceivedFromDisplayNameAtPathMethod];
    if ([title containsString:key]) {
        [self setWantsLayer:YES];
        CALayer *cl = [[CALayer alloc] init];
        [cl setFrame:self.bounds];
        NSColor *highlightColor = [NSColor colorWithRed:166.0 / 255.0 green:1.0 blue:174.0 / 255 alpha:0.2];
        [cl setBackgroundColor:highlightColor.CGColor];
        
        NSArray *subcl = self.layer.sublayers;
        if (subcl != nil) {
            for (CALayer *someLayer in subcl) {
                [someLayer removeFromSuperlayer];
            }
        }
        [self.layer addSublayer:cl];
    } else {
        NSArray *subcl = self.layer.sublayers;
        if (subcl != nil) {
            for (CALayer *someLayer in subcl) {
                [someLayer removeFromSuperlayer];
            }
        }
    }
    
    NSString *processedTitle = [title stringByReplacingOccurrencesOfString:key withString:@""];
    [self new_setTitle:processedTitle];
}

@end

```
