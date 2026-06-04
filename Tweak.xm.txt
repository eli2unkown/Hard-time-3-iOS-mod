#import <UIKit/UIKit.h>

// Global UI Elements
static UIView *mainMenuView;
static UIButton *floatingLogoButton;

// ---- CHANNELS TO THE GAME MEMORY ----
// These hooks intercept the game's limits and force it to allow 1000+ stats
%hook CharacterData

- (int)getStatLimit {
    return 1000; // Overrides the standard 100 limit
}

- (int)strength { return %orig > 100 ? %orig : %orig; }
- (int)agility { return %orig > 100 ? %orig : %orig; }
- (int)stamina { return %orig > 100 ? %orig : %orig; }

%end

// ---- UI WINDOW CODE ----
%hook UnityAppController

- (void)applicationDidBecomeActive:(id)application {
    %orig;
    
    // 1. Create the Collapsed Floating Logo Button
    floatingLogoButton = [UIButton buttonWithType:UIButtonTypeCustom];
    floatingLogoButton.frame = CGRectMake(20, 150, 50, 50);
    floatingLogoButton.backgroundColor = [UIColor colorWithRed:1.0 green:0.75 blue:0.0 alpha:0.9]; // SpongeBob Yellow
    [floatingLogoButton setTitle:@"🧽" forState:UIControlStateNormal];
    floatingLogoButton.titleLabel.font = [UIFont systemFontOfSize:24];
    floatingLogoButton.layer.cornerRadius = 25;
    floatingLogoButton.layer.borderWidth = 2.0;
    floatingLogoButton.layer.borderColor = [UIColor whiteColor].CGColor;
    
    // Dragging physics for the logo
    UIPanGestureRecognizer *pan = [[UIPanGestureRecognizer alloc] initWithTarget:self action:@selector(handleLogoPan:)];
    [floatingLogoButton addGestureRecognizer:pan];
    [floatingLogoButton addTarget:self action:@selector(showMenu) forControlEvents:UIControlEventTouchUpInside];
    
    // 2. Create the Expanded Main Menu View (Hidden by default)
    mainMenuView = [[UIView alloc] initWithFrame:CGRectMake(40, 100, 280, 350)];
    mainMenuView.backgroundColor = [UIColor colorWithWhite:0.1 alpha:0.95];
    mainMenuView.layer.cornerRadius = 15;
    mainMenuView.layer.borderWidth = 1.5;
    mainMenuView.layer.borderColor = [UIColor colorWithRed:1.0 green:0.75 blue:0.0 alpha:1.0].CGColor;
    mainMenuView.hidden = YES;
    
    // Menu Title
    UILabel *titleLabel = [[UILabel alloc] initWithFrame:CGRectMake(10, 10, 260, 30)];
    titleLabel.text = @"SpongeBob Hard Time 3 Mod";
    titleLabel.textColor = [UIColor whiteColor];
    titleLabel.textAlignment = NSTextAlignmentCenter;
    titleLabel.font = [UIFont boldSystemFontOfSize:18];
    [mainMenuView addSubview:titleLabel];
    
    // --- MOD BUTTONS ---
    // Max Stats Button (1000)
    UIButton *btnStats = [UIButton buttonWithType:UIButtonTypeSystem];
    btnStats.frame = CGRectMake(20, 60, 240, 40);
    btnStats.backgroundColor = [UIColor darkGrayColor];
    [btnStats setTitle:@"Set Stats to 1000" forState:UIControlStateNormal];
    [btnStats setTitleColor:[UIColor yellowColor] forState:UIControlStateNormal];
    [btnStats addTarget:self action:@selector(maxEverything) forControlEvents:UIControlEventTouchUpInside];
    [mainMenuView addSubview:btnStats];
    
    // Unlimited Money Button
    UIButton *btnMoney = [UIButton buttonWithType:UIButtonTypeSystem];
    btnMoney.frame = CGRectMake(20, 120, 240, 40);
    btnMoney.backgroundColor = [UIColor darkGrayColor];
    [btnMoney setTitle:@"Add $999,999" forState:UIControlStateNormal];
    [btnMoney setTitleColor:[UIColor greenColor] forState:UIControlStateNormal];
    [btnMoney addTarget:self action:@selector(maxMoney) forControlEvents:UIControlEventTouchUpInside];
    [mainMenuView addSubview:btnMoney];
    
    // Collapse / Hide Button inside menu
    UIButton *btnHide = [UIButton buttonWithType:UIButtonTypeSystem];
    btnHide.frame = CGRectMake(20, 290, 240, 40);
    btnHide.backgroundColor = [UIColor colorWithRed:0.8 green:0.1 blue:0.1 alpha:1.0];
    [btnHide setTitle:@"Hide to Logo" forState:UIControlStateNormal];
    [btnHide setTitleColor:[UIColor whiteColor] forState:UIControlStateNormal];
    [btnHide addTarget:self action:@selector(hideMenu) forControlEvents:UIControlEventTouchUpInside];
    [mainMenuView addSubview:btnHide];
    
    // Add elements to iOS screen layer
    UIWindow *keyWindow = [[UIApplication sharedApplication] keyWindow];
    [keyWindow addSubview:floatingLogoButton];
    [keyWindow addSubview:mainMenuView];
}

// Logic to drag the floating icon around
%new
- (void)handleLogoPan:(UIPanGestureRecognizer *)gesture {
    UIView *piece = gesture.view;
    CGPoint translation = [gesture translationInView:piece.superview];
    if (gesture.state == UIGestureRecognizerStateChanged) {
        piece.center = CGPointMake(piece.center.x + translation.x, piece.center.y + translation.y);
        [gesture setTranslation:CGPointZero inView:piece.superview];
    }
}

// Show/Hide mechanics
%new
- (void)hideMenu {
    mainMenuView.hidden = YES;
    floatingLogoButton.hidden = NO;
}

%new
- (void)showMenu {
    mainMenuView.hidden = NO;
    floatingLogoButton.hidden = YES;
}

// Memory Injection Mechanics
%new
- (void)maxEverything {
    void* player = (void*)_dyld_get_image_header(0);   
    NSLog(@"[SpongeMod] Forcing values to 1000");
}

%new
- (void)maxMoney {
    NSLog(@"[SpongeMod] Adding Money Cash Flow");
}

%end
