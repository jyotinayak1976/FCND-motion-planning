# FCND-Motion-Planning
2nd Project - 3D Motion Planning  

[Detailed Project Explantion Page](https://github.com/udacity/FCND-Motion-Planning)

## Explaning the Starter Code

![Stater_Code](./image/Starter_Code.png)
---

### plan_path() function

Breafly, this function is used to create a configuration space given a map of the world and setting a particular altitude and safety distance for your drone and by using A* algorithm , finding the lowest cost path from start to goal which is used to generates waypoints and send them to simulator.The steps are explained below   

* **1- Read Global Home , Global Position and Local Position**
```
print('global home {0}, position {1}, local position {2}'.format(self.global_home, self.global_position,
                                                                         self.local_position))
```
* **2- Read "Collider.csv" file and obtaining obstacle in the map**
```
data = np.loadtxt('colliders.csv', delimiter=',', dtype='Float64', skiprows=2)
```    
* **3- Creat Grid with a particular altitude and safety margin around obstacles via Planning_utils.py** 
```
grid, north_offset, east_offset = create_grid(data, TARGET_ALTITUDE, SAFETY_DISTANCE)

```  
      * a-Find North_min , North_max , East_min and East_max
      * b-Find Nort and East Size 
      * c-Creat a zero grid array by using Nort and East Size  
      * d-Find obstacles and insert into the grid array
      * e-Return Grid , north_offset , east_offset

Configuration Space 

![Config_Space](./image/Config_Space.png)

* **4- Define Start and Goal Point**
```    
 grid_start = (-north_offset, -east_offset)
       
 grid_goal = (-north_offset + 10, -east_offset + 10)
```
* **5- Run A Star Search Algorithm to find path via Planning_utils.py**  
  
  A* algoritm is used to search the free space for the lowest cost path between the start and the goal.   
  Function inputs are given below.
       
  ```    
  path, path_cost = a_star(grid, heuristic, grid_start, grid_goal)
    
   where
   
   Grid ( Obtian from Step 3 ) , Heuristic ( heuristic function ) , Start ( grid_start ) , Goal  ( grid_goal )
    
  ```
  
  **Steps**
  
   * a - Define a path array , queue (set start point) , visited array ( set start point ) , branch , path cost    
   * b - Check current_node. 
        * If current_node is equel to goal_node , stop search and go **Step e** 
        * If current_node is not equel to goal_node , continue to search
   * c - Find next_node and calculate new_cost 
       
     ```    
     next_node = (current_node[0] + a.delta[0], current_node[1] + a.delta[1])
     new_cost = current_cost + a.cost + heuristic_func(next_node, goal)
  
     ```
     heuristic_func(next_node, goal_node)
     Determines the value for each node based on the goal_node by using the Euclidean method.
     np.linalg.norm(np.array(next_node) - np.array(goal_node)) 
        
   * d - Check next_node.
        * If next_node is not visited,add visisted list, put queue and add into brach
        * If next_node is visited , skip this node and go **Step b**
       
   * e - If a path found . Retrace the path from goal_node to start_node 
   * g - Return Path from start_node to goal_node
   * f - If a path not found . Print (Failed to find a path!) . 
      
      
* **6-Create Waypoint List**

```    
   waypoints = [[p[0] + north_offset, p[1] + east_offset, TARGET_ALTITUDE, 0] for p in path]
```

* **7- Send Waypoint List to Waypoint Array**

```    
  self.waypoints = waypoints
```

* **8- Send Waypoint List to Simulator**

```    
  self.send_waypoints()
```
---

## Implementing Path Planning Algorithm

#### 1. Set your global home position

 * a - Read "collider.csv" file and assign a 'read_pos' string array --> --> [ lat0 37.792480, lon0 -122.397450 ]
 * b - Remove ',' --> [ lat0 37.792480 lon0 -122.397450 ]
 * c - Split String --> [ [lat0], [37.792480], [lon0], [-122.397450] ]
 * d - Assign Lon & Lat Values to self.global_home[0] & self.global_home[1]
```    
  # read file
        from itertools import islice
        filename = 'colliders.csv'  
        with open(filename) as f:
            for line in islice(f, 1):
                read_pos = line

        read_pos = read_pos.replace(",", "") # remove ','
        read_pos = read_pos.split() # split string
       
        self.global_home[0] = float(read_pos[3]) # lon  
        self.global_home[1] = float(read_pos[1]) # lat
        self.global_home[2] = 0
```
#### 2. Set your current local position


```    
  current_local_position = []
  current_local_position = global_to_local (self.global_position, self.global_home)
```


#### 3. Set grid start position from local position

```    
        north_start = int(current_local_position[0])
        easth_start = int(current_local_position[1])

        grid_start = ( (north_start + -north_offset) , (easth_start + -east_offset) )

        print("north_start:",north_start,"easth_start:",easth_start)
        print ("Grid_Start:",grid_start)
```

#### 4. Set grid goal position from geodetic coords

```     #Goal One
        goal_lon = -122.397745 # Desired Goal Position's Longtitude
        goal_lat =  37.793837  # Desired Goal Position's Latitude
        
        goal_pos_global = []
        goal_pos_global = [ goal_lon , goal_lat , 0]

        goal_pos_local = []       
        goal_pos_local = global_to_local (goal_pos_global,self.global_home)
         
        north_goal = int(goal_pos_local[0])
        easth_goal = int(goal_pos_local[1])
        
        grid_goal = ( ( north_goal + -north_offset )  , (easth_goal + -east_offset) )
       
        print("north_stop:",north_goal,"easth_start:",easth_goal)
        print ("Grid_Goal:",grid_goal)
```

#### 5. Modify A* to include diagonal motion (or replace A* altogether)

### planing_utils_sol.py 

Add Diagonal Motion Cost into Action class
```  
    NORTH_WEST = (-1, -1, np.sqrt(2))
    NORTH_EAST = (-1, 1, np.sqrt(2))
    SOUTH_WEST = (1, -1, np.sqrt(2))
    SOUTH_EAST = (1, 1, np.sqrt(2))
        
```  

Also Check Obstacle for Diagonal Motion
```  
    if (x - 1 < 0 or y - 1 < 0) or grid[x - 1, y - 1] == 1:
        valid_actions.remove(Action.NORTH_WEST)
    if (x - 1 < 0 or y + 1 > m) or grid[x - 1, y + 1] == 1:
        valid_actions.remove(Action.NORTH_EAST)
    if (x + 1 > n or y - 1 < 0) or grid[x + 1, y - 1] == 1:
        valid_actions.remove(Action.SOUTH_WEST)
    if (x + 1 > n or y + 1 > m) or grid[x + 1, y + 1] == 1:
        valid_actions.remove(Action.SOUTH_EAST)
        
``` 

#### 6. Cull waypoints

By the help of the Collinearty Check Method [Lecter_6], unnecessary waypoints in path is to eliminate. Breifly, three points p_1, p_2p,p_3 to be collinear, the determinant of the matrix that includes the coordinates of these three points as rows must be equal to zero in three dimension ( necessary but not sufficient) [Detail](http://mathworld.wolfram.com/Collinear.html). 
However in two dimension,z coordinate simply set to 1 and the determinant being equal to zero indicates that the area of the triangle is zero. It is a sufficient condition for collinearity.

In motion_planning_sol.py, prune_path() function is used to eliminate unnecessary waypoints

```     
        from planning_utils import prune_path
        pruned_path = prune_path(path) # path prune

        # Convert path to waypoints
        waypoints = [[p[0] + north_offset, p[1] + east_offset, TARGET_ALTITUDE, 0] for p in pruned_path]
        
```

In planing_utils.py , the details of the prune_path() function is given below

**Details**
   * a - Obtain points p1 , p2 , p3 
   * b - Set z coordinate 1  by the help of the point(p) function
   * c - Check collinearty of p1, p2, p3 by the help of the collinearity_check(p1, p2, p3) function
        * If those points are collinear , remove from pruned_path
        * If those point are not collinear , shift one point
   * d - Return pruned_path
   
``` 
    def prune_path(path):
      pruned_path = [p for p in path]
    
      i = 0
      while i < len(pruned_path) - 2:
        p1 = point(pruned_path[i])
        p2 = point(pruned_path[i+1])
        p3 = point(pruned_path[i+2])
        
      
        if collinearity_check(p1, p2, p3):
            pruned_path.remove(pruned_path[i+1])
        else:
            i += 1
    return pruned_path
```

point(p) -> function is used to set z coordinate to 1
``` 
  def point(p):
    return np.array([p[0], p[1], 1.]).reshape(1, -1)
``` 

collinearty_check (p1,p2,p3,epislon = 1e-2) -> function is used to check collinearty of three points.

 ``` 
 def collinearity_check(p1, p2, p3, epsilon=1e-2): 
 ```

By using [numpy.concatenate](https://docs.scipy.org/doc/numpy-1.14.0/reference/generated/numpy.concatenate.html) function which is used to join two or more arrays of the same shape along a specified axis and [numpy.linalg.det](https://docs.scipy.org/doc/numpy-1.14.0/reference/generated/numpy.linalg.det.html), obtaioned determinant of three points.

 ```
    m = np.concatenate((p1, p2, p3), 0)
    det = np.linalg.det(m)
    return abs(det) < epsilon
 ```    

Epsilon, which indicates how close to zero the determinant must be in order to consider the points to be collinear. This allows you to impose a criterion for accepting points that are almost collinear.

Compare the absolute value of the determinant with epsilon. If the absolute value of determinant is smaller than epsilon , collinearty is true.
 
  ```
  return abs(det) < epsilon
  ```    

 
### Execute the flight

Video

[![Video](https://img.youtube.com/vi/8QoKp-xedlo/0.jpg)](https://www.youtube.com/watch?v=8QoKp-xedlo&feature=youtu.be)

Photos
![Motion_Planning_1](./image/Motion_Planning_1.png)

---

![Motion_Planning_2](./image/Motion_Planning_2.png)



