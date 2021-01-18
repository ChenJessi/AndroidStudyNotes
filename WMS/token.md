# token



- \frameworks\base\services\core\java\com\android\server\wm\AppWindowToken.java


```java
class AppWindowToken extends WindowToken implements WindowManagerService.AppFreezeListener {
    private static final String TAG = TAG_WITH_CLASS_NAME ? "AppWindowToken" : TAG_WM;

    /**
     * Value to increment the z-layer when boosting a layer during animations. BOOST in l33tsp34k.
     */
    private static final int Z_BOOST_BASE = 800570000;
    
    // Non-null only for application tokens.
    final IApplicationToken appToken;//是个Binder
    
     AppWindowToken(WindowManagerService service, IApplicationToken token, boolean voiceInteraction,
            DisplayContent dc, long inputDispatchingTimeoutNanos, boolean fullscreen,
            boolean showForAllUsers, int targetSdk, int orientation, int rotationAnimationHint,
            int configChanges, boolean launchTaskBehind, boolean alwaysFocusable,
            AppWindowContainerController controller) {
        this(service, token, voiceInteraction, dc, fullscreen);
        ...
    }

    AppWindowToken(WindowManagerService service, IApplicationToken token, boolean voiceInteraction,
            DisplayContent dc, boolean fillsParent) {
        super(service, token != null ? token.asBinder() : null, TYPE_APPLICATION, true, dc,
                false /* ownerCanManageAppTokens */);
        appToken = token;
        mVoiceInteraction = voiceInteraction;
        mFillsParent = fillsParent;
        mInputApplicationHandle = new InputApplicationHandle(this);
    }
}
```
appToken是一个Binder
```aidl
/* //device/java/android/android/view/IApplicationToken.aidl
**
** Copyright 2007, The Android Open Source Project
**
** Licensed under the Apache License, Version 2.0 (the "License"); 
** you may not use this file except in compliance with the License. 
** You may obtain a copy of the License at 
**
**     http://www.apache.org/licenses/LICENSE-2.0 
**
** Unless required by applicable law or agreed to in writing, software 
** distributed under the License is distributed on an "AS IS" BASIS, 
** WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. 
** See the License for the specific language governing permissions and 
** limitations under the License.
*/

package android.view;

/** {@hide} */
interface IApplicationToken
{
  String getName();
}
```

WindowToken
```java
class WindowToken extends WindowContainer<WindowState> {
    private static final String TAG = TAG_WITH_CLASS_NAME ? "WindowToken" : TAG_WM;

    // The actual token.
    final IBinder token;

    // The type of window this token is for, as per WindowManager.LayoutParams.
    final int windowType;
    
private final Comparator<WindowState> mWindowComparator =
            (WindowState newWindow, WindowState existingWindow) -> {
        final WindowToken token = WindowToken.this;
        if (newWindow.mToken != token) {
            throw new IllegalArgumentException("newWindow=" + newWindow
                    + " is not a child of token=" + token);
        }

        if (existingWindow.mToken != token) {
            throw new IllegalArgumentException("existingWindow=" + existingWindow
                    + " is not a child of token=" + token);
        }

        return isFirstChildWindowGreaterThanSecond(newWindow, existingWindow) ? 1 : -1;
    };
    
WindowToken(WindowManagerService service, IBinder _token, int type, boolean persistOnEmpty,
            DisplayContent dc, boolean ownerCanManageAppTokens) {
        this(service, _token, type, persistOnEmpty, dc, ownerCanManageAppTokens,
                false /* roundedCornersOverlay */);
    }

    WindowToken(WindowManagerService service, IBinder _token, int type, boolean persistOnEmpty,
            DisplayContent dc, boolean ownerCanManageAppTokens, boolean roundedCornerOverlay) {
        super(service);
        token = _token;
        windowType = type;
        mPersistOnEmpty = persistOnEmpty;
        mOwnerCanManageAppTokens = ownerCanManageAppTokens;
        mRoundedCornerOverlay = roundedCornerOverlay;
        onDisplayChanged(dc);
    }
    
/**
     * Returns true if the new window is considered greater than the existing window in terms of
     * z-order.
     */
protected boolean isFirstChildWindowGreaterThanSecond(WindowState newWindow,
            WindowState existingWindow) {
        // New window is considered greater if it has a higher or equal base layer.
        return newWindow.mBaseLayer >= existingWindow.mBaseLayer;
    }    
    
```





WindowContainer


class WindowContainer<E extends WindowContainer> extends ConfigurationContainer<E>
        implements Comparable<WindowContainer>, Animatable {
     
 // List of children for this window container. List is in z-order as the children appear on
// screen with the top-most window container at the tail of the list.
protected final WindowList<E> mChildren = new WindowList<E>();
     
WindowContainer(WindowManagerService service) {
        mService = service;
        mPendingTransaction = service.mTransactionFactory.make();
        mSurfaceAnimator = new SurfaceAnimator(this, this::onAnimationFinished, service);
    }

/**
     * Adds the input window container has a child of this container in order based on the input
     * comparator.
     * @param child The window container to add as a child of this window container.
     * @param comparator Comparator to use in determining the position the child should be added to.
     *                   If null, the child will be added to the top.
*/
@CallSuper
protected void addChild(E child, Comparator<E> comparator) {
        if (child.getParent() != null) {
            throw new IllegalArgumentException("addChild: container=" + child.getName()
                    + " is already a child of container=" + child.getParent().getName()
                    + " can't add to container=" + getName());
        }

        int positionToAdd = -1;
        if (comparator != null) {
            final int count = mChildren.size();
            for (int i = 0; i < count; i++) {
                if (comparator.compare(child, mChildren.get(i)) < 0) {
                    positionToAdd = i;
                    break;
                }
            }
        }

        if (positionToAdd == -1) {
            mChildren.add(child);
        } else {
            mChildren.add(positionToAdd, child);
        }
        onChildAdded(child);

        // Set the parent after we've actually added a child in case a subclass depends on this.
        child.setParent(this);
    }
    
/**
     * Returns 1, 0, or -1 depending on if this container is greater than, equal to, or lesser than
     * the input container in terms of z-order.
*/
@Override
public int compareTo(WindowContainer other) {
        if (this == other) {
            return 0;
        }

        if (mParent != null && mParent == other.mParent) {
            final WindowList<WindowContainer> list = mParent.mChildren;
            return list.indexOf(this) > list.indexOf(other) ? 1 : -1;
        }

        final LinkedList<WindowContainer> thisParentChain = mTmpChain1;
        final LinkedList<WindowContainer> otherParentChain = mTmpChain2;
        try {
            getParents(thisParentChain);
            other.getParents(otherParentChain);

            // Find the common ancestor of both containers.
            WindowContainer commonAncestor = null;
            WindowContainer thisTop = thisParentChain.peekLast();
            WindowContainer otherTop = otherParentChain.peekLast();
            while (thisTop != null && otherTop != null && thisTop == otherTop) {
                commonAncestor = thisParentChain.removeLast();
                otherParentChain.removeLast();
                thisTop = thisParentChain.peekLast();
                otherTop = otherParentChain.peekLast();
            }

            // Containers don't belong to the same hierarchy???
            if (commonAncestor == null) {
                throw new IllegalArgumentException("No in the same hierarchy this="
                        + thisParentChain + " other=" + otherParentChain);
            }

            // Children are always considered greater than their parents, so if one of the containers
            // we are comparing it the parent of the other then whichever is the child is greater.
            if (commonAncestor == this) {
                return -1;
            } else if (commonAncestor == other) {
                return 1;
            }

            // The position of the first non-common ancestor in the common ancestor list determines
            // which is greater the which.
            final WindowList<WindowContainer> list = commonAncestor.mChildren;
            return list.indexOf(thisParentChain.peekLast()) > list.indexOf(otherParentChain.peekLast())
                    ? 1 : -1;
        } finally {
            mTmpChain1.clear();
            mTmpChain2.clear();
        }
    }
}


```java
/**
 * Contains common logic for classes that have override configurations and are organized in a
 * hierarchy.
 */
public abstract class ConfigurationContainer<E extends ConfigurationContainer> {

    abstract protected int getChildCount();

    abstract protected E getChildAt(int index);

    abstract protected ConfigurationContainer getParent();    
}

```
\frameworks\base\services\core\java\com\android\server\wm\SplashScreenStartingData.java
```java
/**
 * Represents starting data for splash screens, i.e. "traditional" starting windows.
 */
class SplashScreenStartingData extends StartingData {

    private final String mPkg;
    private final int mTheme;
    private final CompatibilityInfo mCompatInfo;
    private final CharSequence mNonLocalizedLabel;
    private final int mLabelRes;
    private final int mIcon;
    private final int mLogo;
    private final int mWindowFlags;
    private final Configuration mMergedOverrideConfiguration;

    SplashScreenStartingData(WindowManagerService service, String pkg, int theme,
            CompatibilityInfo compatInfo, CharSequence nonLocalizedLabel, int labelRes, int icon,
            int logo, int windowFlags, Configuration mergedOverrideConfiguration) {
        super(service);
        mPkg = pkg;
        mTheme = theme;
        mCompatInfo = compatInfo;
        mNonLocalizedLabel = nonLocalizedLabel;
        mLabelRes = labelRes;
        mIcon = icon;
        mLogo = logo;
        mWindowFlags = windowFlags;
        mMergedOverrideConfiguration = mergedOverrideConfiguration;
    }

    @Override
    StartingSurface createStartingSurface(AppWindowToken atoken) {
        return mService.mPolicy.addSplashScreen(atoken.token, mPkg, mTheme, mCompatInfo,
                mNonLocalizedLabel, mLabelRes, mIcon, mLogo, mWindowFlags,
                mMergedOverrideConfiguration, atoken.getDisplayContent().getDisplayId());
    }
}

```



