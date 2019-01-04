
Happy New Year! It's going to be an awesome 2019. I can feel it!

We start out the new year with a challenge I had been discussing with some colleagues:

**Let's build KNN from scratch without using SkLearn or any pre-built libraries.**

A debate ensued as to whether numpy would be allowed, but in the end I decided I would have three methods of doing this: vanilla, numpy, and scipy. Additionally, I am a fan of one-liners. I know, I know, they can get pretty long and un-pythonic. But I think they are fun, so I do them anyway. In the end we have 6 ways of doing KNN. Using the three previously mentioned approaches both as one-liners and non-one-liners.

##### Setup

We will use numpy to initialize the list of points that we will find the nearest neighbors for. That is not to say we are cheating as we are only setting up some random state. In the real world the 'neighbors' can come in many formats and here we are simply initializing.

We also import scipy for the 'euclidean' approach to this solution.

The 'kn' variable determines how many nearest neighbors we are looking for, while 'npts' is how many points we have on our list to check against.

The 'dims' variable is how many dimensions we want our points to be, again, this will be determined by your real data, but here we allow to test for data among any number of dimensions.

While I create the random data using numpy I actually need two versions of the data for the numpy and non-numpy methods. This is why there is a np_pts / pts and np_newpt / newpt. It's the same data, just cast as a numpy array and a list.


```python
import numpy as np
from scipy.spatial.distance import euclidean
kn = 3
npts = 100
dims = 10
np_pts = [np.random.randint(1,100,dims) for i in range(npts)]
pts = [list(pt) for pt in np_pts]
np_newpt = np.random.randint(1,100,dims)
newpt = list(np_newpt)
print("New Point as a Numpy Array:",np_newpt)
print("New Point as a List:",newpt)
```

    New Point as a Numpy Array: [39  6 68 48 14 76 57  5  8 55]
    New Point as a List: [39, 6, 68, 48, 14, 76, 57, 5, 8, 55]


Now that our points and new point are initialized let us build out KNN using vanilla, numpy, and scipy.<br>You will notice the pattern will repeat itself using the three methods. The steps are:
* Iterate though the points
* Calculate the distance
* Append the resulting distance and the data point to a distances list
* Sort the newly formed list of distances
* Show only the top n based on our 'kn' variable via list slicing

Specifically when calculating the distance via the vanilla method, we need to take the extra step of resetting the `sums_of_squares` for each point we are iterating through, summing up those squared distances in the second `for` loop, and taking it's square root.

##### Vanilla Python


```python
distances = []
for pt in pts:
    sums_of_squares = 0 
    for i,j in zip(newpt,pt):
        sums_of_squares += (i-j)**2
    dist = sums_of_squares**0.5
    distances.append([dist, pt])
print(kn,"Nearest Neighbors - Vanilla:")
[print(i) for i in sorted(distances)[:kn]];
    
```

    3 Nearest Neighbors - Vanilla:
    [76.50490180374065, [30, 28, 87, 24, 38, 63, 82, 13, 62, 56]]
    [78.8098978555359, [25, 20, 18, 76, 35, 76, 45, 6, 18, 98]]
    [79.58643100428615, [52, 20, 22, 79, 6, 52, 30, 30, 21, 82]]


##### Using Numpy


```python
distances = []
for pt in pts:
    dist = np.linalg.norm(np_newpt-pt)
    distances.append([dist, pt])
print(kn,"Nearest Neighbors - Numpy:")
[print(i) for i in sorted(distances)[:kn]];
```

    3 Nearest Neighbors - Numpy:
    [76.50490180374065, [30, 28, 87, 24, 38, 63, 82, 13, 62, 56]]
    [78.8098978555359, [25, 20, 18, 76, 35, 76, 45, 6, 18, 98]]
    [79.58643100428615, [52, 20, 22, 79, 6, 52, 30, 30, 21, 82]]


##### Using Scipy


```python
distances = []
for pt in pts:
    dist = euclidean(newpt,pt)
    distances.append([dist, pt])
print(kn,"Nearest Neighbors - Scipy:")
[print(i) for i in sorted(distances)[:kn]];
```

    3 Nearest Neighbors - Scipy:
    [76.50490180374065, [30, 28, 87, 24, 38, 63, 82, 13, 62, 56]]
    [78.8098978555359, [25, 20, 18, 76, 35, 76, 45, 6, 18, 98]]
    [79.58643100428615, [52, 20, 22, 79, 6, 52, 30, 30, 21, 82]]


##### One-Liners

As previously mentioned, I'm a fan of one-liners. So having performed the exercise of building KNN from scratch using the approaches above, we can take these `for` loops and convert them into list comprehensions.

##### Vanilla Python


```python
vanilla_one_liner = sorted([[sum([(i-j)**2 for i, j in zip(newpt,pt)])**0.5, pt] for pt in pts])[:kn]
print(kn,"Nearest Neighbors - Vanilla One-Liner:")
[print(i) for i in vanilla_one_liner];
```

    3 Nearest Neighbors - Vanilla One-Liner:
    [76.50490180374065, [30, 28, 87, 24, 38, 63, 82, 13, 62, 56]]
    [78.8098978555359, [25, 20, 18, 76, 35, 76, 45, 6, 18, 98]]
    [79.58643100428615, [52, 20, 22, 79, 6, 52, 30, 30, 21, 82]]


##### Using Numpy


```python
numpy_one_liner = sorted([[np.linalg.norm(np_newpt-pt), pt] for pt in np_pts])[:kn]
print(kn,"Nearest Neighbors - Numpy One-Liner:")
[print(i) for i in numpy_one_liner];
```

    3 Nearest Neighbors - Numpy One-Liner:
    [76.50490180374065, array([30, 28, 87, 24, 38, 63, 82, 13, 62, 56])]
    [78.8098978555359, array([25, 20, 18, 76, 35, 76, 45,  6, 18, 98])]
    [79.58643100428615, array([52, 20, 22, 79,  6, 52, 30, 30, 21, 82])]


##### Using Scipy


```python
scipy_one_liner = sorted([[euclidean(newpt,pt), pt] for pt in pts])[:kn]
print(kn,"Nearest Neighbors - Scipy One-Liner:")
[print(i) for i in scipy_one_liner];
```

    3 Nearest Neighbors - Scipy One-Liner:
    [76.50490180374065, [30, 28, 87, 24, 38, 63, 82, 13, 62, 56]]
    [78.8098978555359, [25, 20, 18, 76, 35, 76, 45, 6, 18, 98]]
    [79.58643100428615, [52, 20, 22, 79, 6, 52, 30, 30, 21, 82]]


I really enjoyed this exercise. I hope you all enjoyed breaking outside of the canned packaged libraries and getting your hands a little dirty. Thanks for reading, and remember:

Stay Chaotic – Stay Neutral

[ARI](mailto:ari.virrey@gmail.com)
