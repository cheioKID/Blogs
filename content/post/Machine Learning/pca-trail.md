---
date: 2019-01-25
title: "PCA Trail"
tags:
    - Machine Learning
    - SVM
    - PCA
categories:
    - Machine Learning
comment: true
---

## Note

### Prelude

方差\~离散程度\~可区分度

不同投影的损失

successive orthogonal components 连续正交分量，在`fit`方法中学习n个分量

奇异值分解SVD

PCA假定以原点为中心，所以数据需要先中心化*(standard scale)*

```python
from sklearn.preprocessing import StandardScaler

X=StandardScaler().fit(X).transform(X)
# 一般中心化标准化后pca才能体现出效果
```


### Conclusion

经过分析速度得到成倍的提升

PCA 的效果会比 LDA 好一些，还没弄清楚原因

### Error Detail

```
ValueError: n_components=25 must be between 1 and min(n_samples, n_features)=24 with svd_solver='randomized'
```

n_components只能取shape的较小值，是因为SVD吗，现在还不清楚

这种情况下只能提升样本量



## Source Code

```python
# -*- coding:utf-8 -*-

from sklearn.svm import SVC
from skimage import img_as_ubyte
from sklearn.utils import Bunch
import numpy as np
from PIL import Image
import matplotlib.pyplot as plt
from sklearn.preprocessing import StandardScaler
from matplotlib.font_manager import FontProperties
from sklearn.neighbors import KNeighborsClassifier as KNC
from sklearn.model_selection import cross_validate
from mpl_toolkits.mplot3d import Axes3D
from sklearn.decomposition import PCA
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis as LDA
import seaborn as sns
from sklearn.model_selection import train_test_split
from time import time
from sklearn.model_selection import GridSearchCV
from sklearn.metrics import classification_report
from sklearn.metrics import confusion_matrix
from skimage.feature import shape_index


def plot_gallery(images, titles, h=100, w=100, n_row=4, n_col=6):
    """Helper function to plot a gallery of portraits"""
    font = FontProperties(fname='/System/Library/Fonts/STHeiti Light.ttc', size=10)
    plt.figure(figsize=(1.8 * n_col, 2.4 * n_row))
    plt.subplots_adjust(bottom=0, left=.01, right=.99, top=.90, hspace=.35)
    for i in range(n_row * n_col):
        plt.subplot(n_row, n_col, i + 1)
        plt.imshow(images[i].reshape((h, w)), cmap=plt.cm.gray)
        plt.title(titles[i], fontproperties=font)
        plt.xticks(())
        plt.yticks(())


def title_pred(y_pred, y_test, target_names, i):
    pred_name = target_names[y_pred[i]-1]
    true_name = target_names[y_test[i]-1]
    return 'predicted: %s\ntrue:      %s' % (pred_name, true_name)

def title(y_test, target_names, i):
    true_name = target_names[int(y_test[i])-1]
    return true_name

def plot_mse(X,y):
    test_mse=[]
    train_mse=[]

    for n in range(1,X.shape[0]):
        pca=PCA(n_components=n,svd_solver='randomized', whiten=True)
        X_fit_pca=pca.fit(X).transform(X)

        knc=KNC()
        scorer='accuracy'
        print(X_fit_pca.shape, y.shape)
        knc_dict=cross_validate(knc,X_fit_pca,y,cv=5, scoring=scorer, return_train_score=True)

        test_mse.append(knc_dict['test_score'].mean())
        train_mse.append(knc_dict['train_score'].mean())

    sns.set(style='darkgrid')
    print(X.shape, len(test_mse), len(train_mse))

    plt.plot(range(1,X.shape[0]),test_mse,'b-',label='test mse')
    plt.plot(range(1,X.shape[0]),train_mse,'r-',label='train mse')
    plt.xlabel('Dimensions')
    plt.ylabel('MSE')

    plt.legend()



def load_samples(path, slen=120, names=['白菜','白梨','白萝卜','板栗','包菜','本地黑李','本地红提','本地黄瓜','菠萝','菜瓜','菜心','春菜']):
    y = []
    images = np.zeros((slen, 100, 100), dtype=np.float32) # (24, 100, 100)

    i=0
    f_fruits = open(path,'r')
    for fruits in f_fruits:
        img_path = path[:-4] + '/' + fruits.split(" ")[0]
        img = Image.open(img_path).convert('L')		#get gray img
        # shape_img = shape_index(img, sigma = 0.1)
        # color_img = img_as_ubyte(color_img)			#change from float64 to uint8
        images[i] = img
        i+=1

        label = fruits.split(" ")[1]
        y.append(int(label.split("\n")[0]))

    feature_color = images.reshape(len(images), -1) # (24, 10000)

    return Bunch(data=feature_color, images=images,
                 target=np.array(y), target_names=names)


def with_pca(X_train, X_test, y_train, h=100, w=100):
    ####################
    # pca

    n_components = 6

    print("Extracting the top %d samples from %d samples" % (n_components, X_train.shape[0]))
    pca = PCA(n_components=n_components, svd_solver='randomized', whiten=True).fit(X_train)

    images_pca_reshaped = pca.components_.reshape((n_components, h, w))

    X_train_pca = pca.transform(X_train)
    X_test_pca = pca.transform(X_test)

    ####################
    # grid search
    print("Fitting the classifier to the training set")
    t0 = time()
    param_grid = {'C': [1e3, 5e3, 1e4, 5e4, 1e5],
                  'gamma': [0.0001, 0.0005, 0.001, 0.005, 0.01, 0.1], }
    clf = GridSearchCV(SVC(kernel='rbf', class_weight='balanced'),
                       param_grid, cv=5)
    clf = clf.fit(X_train_pca, y_train)
    print("done in %0.3fs" % (time() - t0))
    print("Best estimator found by grid search:")
    print(clf.best_estimator_)

    return clf, X_test_pca


def with_lda(X_train, X_test, y_train, h=100, w=100):
    ####################
    # pca

    n_components = 6

    print("Extracting the top %d samples from %d samples" % (n_components, X_train.shape[0]))
    lda = LDA(n_components=n_components).fit(X_train, y_train)

    # images_pca_reshaped = lda.components_.reshape((n_components, h, w))

    X_train_pca = lda.transform(X_train)
    X_test_pca = lda.transform(X_test)

    ####################
    # grid search
    print("Fitting the classifier to the training set")
    t0 = time()
    param_grid = {'C': [1e3, 5e3, 1e4, 5e4, 1e5],
                  'gamma': [0.0001, 0.0005, 0.001, 0.005, 0.01, 0.1], }
    clf = GridSearchCV(SVC(kernel='rbf', class_weight='balanced'),
                       param_grid, cv=5)
    clf = clf.fit(X_train_pca, y_train)
    print("done in %0.3fs" % (time() - t0))
    print("Best estimator found by grid search:")
    print(clf.best_estimator_)

    return clf, X_test_pca

def without_decomposition(X_train, y_train):
    print("Fitting the classifier to the training set")
    t0 = time()
    param_grid = {'C': [1e3, 5e3, 1e4, 5e4, 1e5],
                  'gamma': [0.0001, 0.0005, 0.001, 0.005, 0.01, 0.1], }
    clf = GridSearchCV(SVC(kernel='rbf', class_weight='balanced'),
                       param_grid, cv=5)
    clf = clf.fit(X_train, y_train)
    print("done in %0.3fs" % (time() - t0))
    print("Best estimator found by grid search:")
    print(clf.best_estimator_)

    return clf



train_file = '../fruit_recognize_samples/train.txt'
test_file = 'fruit_recognize_samples/test.txt'

samples = load_samples(train_file)

X = samples.data
n_features = X.shape[1]

y=samples.target
target_names = samples.target_names
n_classes = len(target_names)

n_samples, h, w = samples.images.shape

print("Total dataset size:")
print("n_samples: %d" % n_samples)
print("n_features: %d" % n_features)
print("n_classes: %d" % n_classes)


X=StandardScaler().fit(X).transform(X)

#plot_mse(X,y)
#plt.show()

####################
# split
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.25, random_state=42)


# plot_mse(X_train,y_train)

############################################################
# with PCA

t0 = time()
clf, X_test_pca = with_pca(X_train, X_test, y_train)

# predict

print("Predicting fruits on the test set with pca")
t0 = time()
y_pred = clf.predict(X_test_pca)
print("predict done in %0.3fs" % (time() - t0))

print(y_test)
print(y_pred)
print(classification_report(y_test, y_pred, target_names=target_names))
# print(confusion_matrix(y_test, y_pred, labels=range(n_classes)))

titles = [title_pred(y_pred, y_test, target_names, i) for i in range(len(y_test))]
plot_gallery(X_test, titles, n_row=3, n_col=10)
############################################################

print("\n\n\n\n\n\n")

############################################################
# with LDA

t0 = time()
clf, X_test_pca = with_lda(X_train, X_test, y_train)

# predict

print("Predicting fruits on the test set with lda")
t0 = time()
y_pred = clf.predict(X_test_pca)
print("predict done in %0.3fs" % (time() - t0))

print(y_test)
print(y_pred)
print(classification_report(y_test, y_pred, target_names=target_names))
# print(confusion_matrix(y_test, y_pred, labels=range(n_classes)))

titles = [title_pred(y_pred, y_test, target_names, i) for i in range(len(y_test))]
plot_gallery(X_test, titles, n_row=3, n_col=10)
############################################################

print("\n\n\n\n\n\n")

############################################################
# without PCA

t0 = time()
clf = without_decomposition(X_train, y_train)
print("without pca done in %0.3fs" % (time() - t0))

# predict

print("Predicting fruits on the test set without pca")
t0 = time()
y_pred = clf.predict(X_test)
print("predict done in %0.3fs" % (time() - t0))

print(y_test)
print(y_pred)
print(classification_report(y_test, y_pred, target_names=target_names))

titles = [title_pred(y_pred, y_test, target_names, i) for i in range(len(y_test))]
plot_gallery(X_test, titles, n_row=3, n_col=10)
############################################################



plt.show()

```



## Reference

- https://scikit-learn.org/stable/auto_examples/applications/plot_face_recognition.html#sphx-glr-auto-examples-applications-plot-face-recognition-py
