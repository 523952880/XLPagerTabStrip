# XLPagerTabStrip
Xamarin port of the iOS library [XLPagerTabStrip](https://github.com/xmartlabs/XLPagerTabStrip) by [XMARTLABS](http://xmartlabs.com/).

Android [PagerTabStrip](http://developer.android.com/reference/android/support/v4/view/PagerTabStrip.html) for iOS!

**This is not a binding library. It is a C# port of the Swift version of the library.**

**XLPagerTabStrip** is a _Container View Controller_ that allows us to switch easily among a collection of view controllers. Pan gesture can be used to move on to next or previous view controller. It shows a interactive indicator of the current, previous, next child view controllers.

Dou you use the library? You can check us out on [twitter](https://twitter.com/supersume)

# Pager types
Currently the library supports only one pager type

## Button Bar
This is likely to be the most common pager type. It's used by many well known apps such as instagram, youtube, skype and many others.

![Screenshot](https://github.com/supersume/XLPagerTabStrip/blob/master/Simulator.png?raw=true)

Support for other pager types, `TwitterPagerTabStripViewController`, `SegmentedPagerTabStripViewController`,`BarPagerTabStripViewController` will be provided with future releases.

# Usage
Basically we just need to provide the list of child view controllers to show and these view controllers should provide the information (title or image) that will be shown in the associated indicator.

Let's see the steps to do this:

##### Choose which type of pager we want to create

First we should choose the type of pager we want to create, depending on our choice we will have to create a view controller that inherits from one of the following controllers: `TwitterPagerTabStripViewController`, `ButtonBarPagerTabStripViewController`, `SegmentedPagerTabStripViewController`,`BarPagerTabStripViewController`.

> All these build-in pager controllers extend from the base class PagerTabStripViewController. You can also make your custom pager controller by extending directly from PagerTabStripViewController in case no pager menu type fits your needs.

```c#
using XLPagerTabStrip;

public class MyPagerTabStripName: ButtonBarPagerTabStripViewController 
{
  ..
}
```

##### Provide the view controllers that will appear embedded into the PagerTabStrip view controller

You can provide the view controllers by overriding `CreateViewControllersForPagerTabStrip(PagerTabStripViewController pagerTabStripController)` method.

```c#
public override UIViewController[] CreateViewControllersForPagerTabStrip(PagerTabStripViewController pagerTabStripViewController) 
{
  return new UIViewController[] { MyEmbeddedViewController(), MySecondEmbeddedViewController() };
}
```

> The method above is the only method declared in `IPagerTabStripDataSource` interface. We don't need to explicitly conform to it since base pager class already does it.


##### Provide information to show in each indicator

Every UIViewController that will appear within the PagerTabStrip needs to provide either a title or an image.
In order to do so they should implement the `IIndicatorInfoProvider` interface method `IndicatorInfoForPagerTabStrip(PagerTabStripViewController pagerTabStripController)`
 which provides the information required to show the PagerTabStrip menu (indicator) associated with the view controller.

```c#
public class MyEmbeddedViewController: UITableViewController, IIndicatorInfoProvider
{
  public IndicatorInfo IndicatorInfoForPagerTabStrip(PagerTabStripViewController pagerTabStripController)     {
    return new IndicatorInfo("My Child title");
  }
}
```

## Customization

##### Pager Behaviour

The pager indicator can be updated progressive as we swipe or at once in the middle of the transition between the view controllers.
By setting up `pagerBehaviour` property we can choose how the indicator should be updated.

```c#
public PagerTabStripBehaviour pagerBehaviour;
```

```c#
public class PagerTabStripBehaviour 
{
    public bool? SkipIntermediateViewControllers { get; set; };
    public bool? ElasticIndicatorLimit { get; set; };
}
```

Default Values:
```c#
// Twitter Type
PagerTabStripBehaviour.Create(skipIntermediteViewControllers: true);
// Segmented Type
PagerTabStripBehaviour.Create(skipIntermediteViewControllers: true);
// Bar Type
PagerTabStripBehaviour.Create(skipIntermediteViewControllers: true, elasticIndicatorLimit: true);
// ButtonBar Type
PagerTabStripBehaviour.Create(skipIntermediteViewControllers: true, elasticIndicatorLimit: true);
```

As you might have noticed `PagerTabStripBehaviour` class cases has `SkipIntermediteViewControllers` and `ElasticIndicatorLimit` properties.

`SkipIntermediteViewControllers` allows us to skip intermediate view controllers when a tab indicator is tapped.

`ElasticIndicatorLimit` allows us to tension the indicator when we reach a limit, I mean when we try to move forward from last indicator or move back from first indicator.

##### IPagerTabStripDelegate & IPagerTabStripIsProgressiveDelegate

Normally we don't need to implement these interfaces because each pager type already conforms to it in order to properly update its indicator. Anyway there may be some scenarios when overriding a method come come in handy.

```c#
public interface IPagerTabStripDelegate 
{
    void PagerTabStripViewController(PagerTabStripViewController pagerTabStripViewController, int fromIndex, int toIndex);
}

public interface IPagerTabStripIsProgressiveDelegate : IPagerTabStripDelegate 
{
    void PagerTabStripViewController(PagerTabStripViewController pagerTabStripViewController, int fromIndex, int toIndex, nfloat progressPercentage, bool indexWasChanged);
}
```

> Again, The method invoked by the library depends on the `PagerBehaviour` value.

### ButtonBar Customization

```c#

public UIColor ButtonBarBackgroundColor { get; set; }
// buttonBar minimumLineSpacing value
public nfloat? ButtonBarMinimumLineSpacing { get; set; }
// buttonBar flow layout left content inset value
public nfloat? ButtonBarLeftContentInset { get; set; }
// buttonBar flow layout right content inset value
public nfloat? ButtonBarRightContentInset { get; set; }


// selected bar view is created programmatically so it's important to set up the following 2 properties properly
public UIColor SelectedBarBackgroundColor { get; set; } = UIColor.Black;
public nfloat? SelectedBarHeight { get; set; } = 4;

// each buttonBar item is a UICollectionView cell of type ButtonBarViewCell
public UIColor ButtonBarItemBackgroundColor { get; set; }
public UIFont ButtonBarItemFont { get; set; } = UIFont.SystemFontOfSize(18);
// helps to determine the cell width, it represent the space before and after the title label
public nfloat? ButtonBarItemLeftRightMargin { get; set; } = 8;
public UIColor ButtonBarItemTitleColor { get; set; }
// in case the barView items do not fill the screen width this property stretch the cells to fill the screen
public bool ButtonBarItemsShouldFillAvailiableWidth { get; set; } = false;

// only used if button bar is created programaticaly and not using storyboards or nib files
public nfloat? ButtonBarHeight { get; set; }
```

**Important:** Settings should be called before `ViewDidLoad` is called.
```c#
public override void ViewDidLoad() 
{
   this.Settings.Style.SelectedBarHeight = 2;
   this.Settings.Style.SelectedBarBackgroundColor = UIColor.White;

   base.ViewDidLoad();
}
```

#####  Update cells when selected indicator changes

We may need to update the indicator cell when the displayed view controller changes. The following function properties help to accomplish that. Depending on our pager `PagerBehaviour` property value we will have to set up `ChangeCurrentIndex` or `ChangeCurrentIndexProgressive`.

```c#
public void changeCurrentIndex: (ButtonBarViewCell oldCell, ButtonBarViewCell newCell, bool animated);
public void changeCurrentIndexProgressive(ButtonBarViewCell oldCell, ButtonBarViewCell newCell, nfloat progressPercentage, bool changeCurrentIndex, bool animated);
```

Let's see an example:

```c#
void changeCurrentIndexProgressive(ButtonBarViewCell oldCell, ButtonBarViewCell newCell, nfloat progressPercentage, bool changeCurrentIndex, bool animated)
{
    if (changeCurrentIndex == true)
    {
        if (oldCell != null)
            oldCell.Label.TextColor = UIColor.White;
        if (newCell != null)
            newCell.Label.TextColor = UIColor.Black;
    }
}
```

## FAQ

#### How to change the visible child view controller programmatically

`XLPagerTabStripViewController` provides the following methods to programmatically change the visible child view controller:

```c#
void MoveToViewControllerAtIndex(int index);
void MoveToViewControllerAtIndex(int index, bool animated);
void MoveToViewController(UIViewController viewController);
void MoveToViewController(UIViewController viewController, bool animated);
```

## Author

* [Asumege Alison](https://github.com/supersume) ([@supersume](https://twitter.com/supersume))

## Credits to

* [Martin Barreto](https://github.com/mtnBarreto) ([@mtnBarreto](https://twitter.com/mtnBarreto))
