---
ID: 74
post_title: >
  Creating a two-dimensional scroll
  interface for iOS using UIPageView and
  UICollectionView
author: Lorenzo Bulfone
post_date: 2015-10-14 13:06:31
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2015/10/14/creating-a-two-dimensional-scroll-interface-for/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/131152800359/creating-a-two-dimensional-scroll-interface-for
tumblr_mikamayhem_id:
  - "131152800359"
---
<p>Recently I&rsquo;ve been looking into ways to create an interface that allows the user to navigate through full-page views using both vertical and horizontal scrolling. Each view provides a data chart (replaced by a simple label in this example) for a specific day, week or month. By swiping left/right the user can navigate between days, while swiping up/down they can switch to the weekly (and further down, monthly) view and back up. The navigation works in a very similar way to Sony PS3&rsquo;s and PS4&rsquo;s menu interface.</p><p>iOS provides different ways to display scrolling and paged content, namely <code>UIPageView</code>, <code>UIScrollView</code> and <code>UICollectionView</code>. The last of the three was the last to be introduced and it is the most versatile thanks to the ability to use a custom <code>UICollectionViewLayout</code> to create a grid interface.
At first, a grid interface with only one view at a time on-screen seemed like the way to go, but then the horizontal and vertical navigation would have been tied together: if the user navigates 3 days back in time (horizontally) and then swipes down to the week view, they would be provided with the data from 3 weeks back, and this would be counterintuitive and confusing.</p><p>The solution was to have three different <code>UICollectionView</code> to handle the horizontal navigation independently and a <code>UIPageView</code> to navigate vertically between the three.</p><p>I added a <code>UIPageViewController</code> to the storyboard, and in the Attribute Inspector I set the navigation direction to vertical and the transition style to scroll. This will be the initial view controller.</p><p>Then I added a <code>UICollectionViewController</code> and set its storyboard ID to &ldquo;collectionViewController&rdquo; in the Identity Inspector. I navigated the view hierarchy down one level to the <code>UICollectionView</code> and configured it as follows.</p><figure><img src="http://68.media.tumblr.com/5269f7f62f0c5c96ff30267336bdc2a7/tumblr_inline_nw8g2gttGn1tq7pdk_540.png" /></figure><p>The UIPageView is handled by <code>VerticalPageViewController</code>:</p><pre><code>@implementation VerticalPageViewController

- (void)viewDidLoad
{
    [super viewDidLoad];

    // Set yourself as the datasource and delegate of the page view
    self.dataSource = self;
    self.delegate = self;

    // Get the first of the three UICollectionViewControllers and load it on screen
    HorizontalCollectionViewController *startingViewController = [self viewControllerAtIndex:0];
    NSArray *viewControllers = @[startingViewController];
    [self setViewControllers:viewControllers direction:UIPageViewControllerNavigationDirectionReverse animated:NO completion:nil];
}

#pragma mark - UIPageViewController Delegate &amp; Datasource

- (UIViewController *)pageViewController:(UIPageViewController *)pageViewController viewControllerBeforeViewController:(UIViewController *)viewController
{
    NSInteger index = ((HorizontalCollectionViewController *) viewController).pageIndex;

    if (index == NSNotFound) {
        return nil;
    }

    index--;

    return [self viewControllerAtIndex:index];
}

- (UIViewController *)pageViewController:(UIPageViewController *)pageViewController viewControllerAfterViewController:(UIViewController *)viewController
{
    NSInteger index = ((HorizontalCollectionViewController *) viewController).pageIndex;

    if (index == NSNotFound) {
        return nil;
    }

    index++;

    return [self viewControllerAtIndex:index];
}

- (HorizontalCollectionViewController *)viewControllerAtIndex:(NSInteger)index
{
    if ((index &gt; 0) || (index &lt; -2)) {
        return nil;
    } else {

        // Get a new UICollectionViewController (with all the settings and subviews) from the storyboard
        HorizontalCollectionViewController *collectionViewController = [self.storyboard instantiateViewControllerWithIdentifier:@"collectionViewController"];
        // Give the controller a page index so it is aware of which data to display (day/week/month)
        collectionViewController.pageIndex = index;

        return collectionViewController;
    }
}       

@end
</code></pre><p>The three UICollectionView are handled by a single controller, <code>HorizontalCollectionViewController.m</code>.</p><pre><code>@interface HorizontalCollectionViewController ()

@property IBOutlet UILabel *label;

@property NSMutableArray *dailyContent;
@property NSMutableArray *weeklyContent;
@property NSMutableArray *monthlyContent;

@end

@implementation HorizontalCollectionViewController

static NSString * const reuseIdentifier = @"Cell";

- (void)viewDidLoad
{
    [super viewDidLoad];

    // Register cell classes
    // [self.collectionView registerClass:[UICollectionViewCell class] forCellWithReuseIdentifier:reuseIdentifier];

    // Do any additional setup after loading the view.
    _dailyContent = [[NSMutableArray alloc] initWithArray:@[@"DAY: One", @"DAY: Two", @"DAY: Three", @"DAY: Four", @"DAY: Five"]];
    _weeklyContent = [[NSMutableArray alloc] initWithArray:@[@"WEEK: One", @"WEEK: Two", @"WEEK: Three", @"WEEK: Four", @"WEEK: Five"]];
    _monthlyContent = [[NSMutableArray alloc] initWithArray:@[@"MONTH: One", @"MONTH: Two", @"MONTH: Three", @"MONTH: Four", @"MONTH: Five"]];

    [self scrollToEnd];
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

/*
#pragma mark - Navigation

// In a storyboard-based application, you will often want to do a little preparation before navigation
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender {
    // Get the new view controller using [segue destinationViewController].
    // Pass the selected object to the new view controller.
}
*/

#pragma mark &lt;UICollectionViewDataSource&gt;

- (NSInteger)numberOfSectionsInCollectionView:(UICollectionView *)collectionView
{
    return 1;
}


- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section
{
    // Depending on the page index (0 to -2) figure out which data to display
    switch (_pageIndex)
    {
        case 0: {
            return [_dailyContent count];
            break;
        }

        case -1: {
            return [_weeklyContent count];
            break;
        }


        case -2: {
            return [_monthlyContent count];
            break;
        }

        default: {
            return 1;
            break;
        }
    }
}

- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath
{
    // Don't forget to give the UICollectionViewCell a class and a storyboard ID in the Interface Builder
    // HorizontalCollectionViewCell.h declared a IBOutlet UILabel *label
    // This way the compiler knows that each cell has a label and doesn't complain (wire the outlet to the placeholder cell in the Interface Builder)
    HorizontalCollectionViewCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:@"MovesCell" forIndexPath:indexPath];

    // Get the content from the correct array and update the label
    switch (_pageIndex)
    {
        case 0: {
            cell.label.text = [_dailyContent objectAtIndex:indexPath.row];
            break;
        }

        case -1: {
            cell.label.text = [_weeklyContent objectAtIndex:indexPath.row];
            break;
        }


        case -2: {
            cell.label.text = [_monthlyContent objectAtIndex:indexPath.row];
            break;
        }

        default: {
            break;
        }
    }

    return cell;
}

#pragma mark UICollectionViewDelegate

- (CGSize)collectionView:(UICollectionView *)collectionView layout:(UICollectionViewLayout *)collectionViewLayout sizeForItemAtIndexPath:(NSIndexPath *)indexPath
{
    // Each cell must be the same as the screen size minus the height of the status bar (otherwise it will brake the UICollectionViewFlowLayout)
    CGFloat width = self.view.frame.size.width;
    CGFloat height = self.view.frame.size.height - [UIApplication sharedApplication].statusBarFrame.size.height;

    return CGSizeMake(width, height);
}

@end
</code></pre><p>The header files for these two classes just declare the public properties and the protocols the two classes conform to. Using a custom subclass of UICollectionViewCell allows to declare its outlets and to design its interface easily in the storyboard.</p><p>If everything has been wired and set correctly in the storyboard, these few lines of code will allow you to create a pretty neat and complex UI!</p>