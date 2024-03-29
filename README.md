# Graph filtration

You can also find this page [here](https://sites.google.com/site/hkleebrain/home/persistent-homology-2). 

The method for graph filtration is based on the following papers and abstract: 
- Hyekyoung Lee,  Moo K. Chung, Hyejin Kang, Bung-Nyun Kim, and  Dong Soo Lee (2012),"Persistent brain network homology from the perspective of dendrogram," IEEE Transactions on Medical Imaging. in press. 
- Hyekyoung Lee,  Moo K. Chung, Hyejin Kang, Bung-Nyun Kim, and  Dong Soo Lee (2011), "Computing the shape of brain network using graph filtration and Gromov-Haudorff metric," MICCAI2011, Toronto, Canada, September 18-22, 2011.
- Hyekyoung Lee,  Moo K. Chung, Hyejin Kang, Bung-Nyun Kim, and  Dong Soo Lee  (2010), "Discriminative persistent homology of brain networks," IEEE International Symposium on Biomedical Imaging (ISBI), Chicago, U.S.A., March 30 - April 2, 2011.
- Hyekyoung Lee, Moo K. Chung, Hyejin Kang, Bung-Nyun Kim and Dong Soo Lee (2011), "Persistent network homology from the perspective of dendrograms," 17th Annual Meeting of the Organization for Human Brain Mapping (HBM), Quebec City, Canada, June 26-30, 2011.

--- 


We generate 10 probability maps and sample 100 data points randomly from each map.   

![probability_map](https://user-images.githubusercontent.com/54297018/63376225-429d6e80-c3c8-11e9-8f8c-c07f149a7121.jpg)

We have 200 datasets, 20 from C1, ..., 20 from C10.  Each dataset has 100 data points in the 2-dimensional space.   

```Matlab 
load 'X.mat';
```
  
The size of X is `[100 2 200]`.  


Then, estimate their distance matrices, single linkage matrices, geodesic distance matrices and minimum spanning trees. 

```Matlab 
C = [];              % distance matrix
D = [];              % single linkage matrix
MST = [];          % minimum spanning tree
for cv = 1:200    % for 200 datasets
   C(:,:,cv) = sqrt(distance_between_mtx(X(:,:,cv),X(:,:,cv)));        % estimate the distance between the data points
   [D(:,:,cv),MST(:,:,cv)] = single_linkage_matrix(C(:,:,cv));
   display(num2str([cv]));
end
```

The size of C, D and W are `[100 100 200]` and size of MST is `[99 3 200]`.   
Each row of MST consists of the indices of two nodes, which are connected by an edge, and its edge distance.   
  
  
  
Visualize the distance matrices and single linkage matrices.   

```Matlab
color_list = colormap(hot);
color_list = color_list(end:-1:1,:);
figure; 
for i = 1:10
    subplot(2,10,i); 
    % We plot the 8th dataset among 20 datasets generated by each probability map. 
    cv = 20*(i-1)+8;
    imagesc(C(:,:,cv),[0 670]); colormap(color_list); 
    subplot(2,10,i+10); 
    imagesc(D(:,:,cv),[0 150]); colormap(color_list); 
end
```

![c_d](https://user-images.githubusercontent.com/54297018/63376399-ade74080-c3c8-11e9-989b-853c423c0603.jpg)


The barcode of connected components is a decreasing function depending on the death time of connected components because the birth time of connected components is always zero. 

```Matlab
% The color of barcode representing each probability map.  
S = 230;                        % saturation
V = 220;                        % brightness
H = [255/10:255/10:255]; % hue
color = [];
color = hsv2rgb([H' S*ones(10,1) V*ones(10,1)]/255);

figure;
for i = 1:10
    cv = 20*(i-1)+8;
    plot([0; MST(:,3,cv); 300],[100 [99:-1:1] 1],'Color',color(i,:),'LineWidth',2);
    hold on;
    xlim([0 100]);
    ylim([0 103]);
end
``` 

![barcode](https://user-images.githubusercontent.com/54297018/63376269-63fe5a80-c3c8-11e9-9f22-92015e9d53e6.jpg)  


Depending on the connected structure of each dataset, the barcodes have different shapes. 
We quantify the shape of barcode using the slope and kurtosis. 
They are estimated as follows.   

```Matlab
for cv = 1:200
        [s(cv,1)] = slope_barcode(MST(:,3,cv)); 
        [kurt(cv,1)] = kurtosis_barcode(MST(:,3,cv));
       display(num2str(cv));
end
```


Visualize the kurtosises when the slopes are varied.  

```Matlab
count = 1;
for i = 1:10
    for cv = 1:20
        plot(abs(s(count,1)),kurt(count,1),'o','MarkerEdgeColor','k',...
                'MarkerFaceColor',color(i,:),'MarkerSize',5); % ,'LineWidth',0.5);
        hold on; 
        count = count + 1;
    end
    ylim([0 12]), xlim([0 3.5]);
end
``` 

![slope_kurtosis](https://user-images.githubusercontent.com/54297018/63376347-94de8f80-c3c8-11e9-89f1-2d440bebab07.jpg)


Plot the dendrograms.   

```Matlab
figure;
for i = 1:10
    subplot(1,10,i);
    cv = (i-1)*20+8; 
    max_dist(i) = dendrogram(MST(:,:,cv));
end
```

![dendrograms](https://user-images.githubusercontent.com/54297018/63376312-7ed0cf00-c3c8-11e9-9db9-df370fb453ec.jpg)  


When the correspondence between two node sets is determined, the Gromov-Hausdorff distance is just the maximum distance between the single linkage matrices.   

```Matlab
Dgh = [];
for i = 1:200
    for j = 1:200
        pair = correspondence(X(:,:,i),X(:,:,j));
        Dgh(i,j) = max(max(abs(D(pair(:,1),pair(:,1),i) - D(pair(:,2),pair(:,2),j))));
        display(num2str([i j]));
    end
end
imagesc(Dgh/max(max(Dgh))), colormap(color_list); % the maximum is 1
``` 

![Dgh](https://user-images.githubusercontent.com/54297018/63376428-c192a700-c3c8-11e9-8f9c-97a2b2186baa.jpg)


Based on the GH distance matrix, we cluster 100 single linkage matrices to 10 groups using the Ward's cluster analysis.   

```Matlab
ytrue = repmat([1:10],[20 1]);;
ytrue = reshape(ytrue,20*10,1);
acc_gh = cluster_accuracy(Dgh,ytrue,10);
```

We can also estimate the bottleneck distance and the clustering accuracy of it.    


```Matlab
for i = 1:200
    for j = 1:200
        Dbottle(i,j) = bottleneck_distance([zeros(99,1) MST(:,3,i)],[zeros(99,1) MST(:,3,j)]);
        display(num2str([i j]));
    end
end
acc_bottle = cluster_accuracy(Dbottle,ytrue,10);
```

The clustering accuracy of GH distance is `acc_gh = 0.8700`, and the clustering accuracy of bottleneck distance with the zeroth homology group is `acc_bottle = 0.2800`. 

