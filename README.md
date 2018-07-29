# PoissonDiskSampling

Poisson Disk Sampling algorithm, based on http://www.cs.ubc.ca/~rbridson/docs/bridson-siggraph07-poissondisk.pdf

First implementation (Processing): https://gist.github.com/andrearastelli/eababa41e921349a01748d4babea3da2

```java
import java.awt.*;

ArrayList<Point> points = new ArrayList<Point>();
ArrayList<Point> active_list = new ArrayList<Point>();

int WINDOW = 400;
float radius = 10;
int size = 200;
int rejection = 3;
int size_ratio = WINDOW / size;

int[][] sample_grid = new int[size][size];

void setup()
{
  println(size);
  noStroke();
  fill(0);
  frameRate(60);
  size(400, 400);
  
  for (int i=0; i<size; ++i) for (int j=0; j<size; ++j) sample_grid[i][j] = -1;
  
  Point x0 = new Point((int)random(size*size_ratio), (int)random(size*size_ratio));
  active_list.add(x0);
}

void fill_sample_grid(int[][] sample_grid, Point tmp1)
{
  for (int theta=0; theta<360; ++theta)
  {
    for (int r=0; r<radius/size_ratio; ++r)
    {
      int cx = (int)(tmp1.getX()/size_ratio + r * cos(radians(theta)));
      int cy = (int)(tmp1.getY()/size_ratio + r * sin(radians(theta)));
      if (cx < 0) cx = 0; if (cx >= size) cx = size-1;
      if (cy < 0) cy = 0; if (cy >= size) cy = size-1;
      sample_grid[cx][cy] = 0;
    }
  }
}

Point generate_xy(Point tmp)
{
  int x = -1;
  int y = -1;
  while((x < 0) || (x >= size) || (y < 0) || (y >= size))
  {
    float angle = random(2 * PI);
    float d = (float)random(radius/size_ratio) + radius/size_ratio;
    x = (int)((tmp.getX()/size_ratio)+(d)*cos(angle));
    y = (int)((tmp.getY()/size_ratio)+(d)*sin(angle));
  }
  
  return new Point(x, y);
}

void draw()
{
  background(128);
  noStroke();
  fill(0);
  
  for (int i=0; i<points.size(); ++i)
  {
    Point p = points.get(i);
    ellipse((float)p.getX(), (float)p.getY(), 2, 2);
  }
  
  for (int i=0; i<active_list.size(); ++i)
  {
    Point p = active_list.get(i);
    
    fill(color(100, 100, 100, 60));
    ellipse((float)p.getX(), (float)p.getY(), 2*radius, 2*radius);
    
    fill(color(255, 0, 0, 50));
    ellipse((float)p.getX(), (float)p.getY(), radius, radius);
    
    fill(color(255, 0, 0));
    ellipse((float)p.getX(), (float)p.getY(), 2, 2);
  }
  
  if (active_list.size() > 0)
  {
    Point tmp = active_list.get((int)random(active_list.size()));
    
    fill_sample_grid(sample_grid, tmp);
    
    int active_list_size = active_list.size();
    
    for (int idx=0; idx<rejection; ++idx)
    {
      Point xy = generate_xy(tmp);
      int x = (int)xy.getX();
      int y = (int)xy.getY();
      Point tmp1 = new Point(x*size_ratio, y*size_ratio);
      
      fill(255);
      ellipse((float)tmp1.getX(), (float)tmp1.getY(), 1, 1);
      stroke(255, 120);
      line((float)tmp.getX(), (float)tmp.getY(), (float)tmp1.getX(), (float)tmp1.getY());
      
      if (sample_grid[x][y] != -1) continue;
      else
      {
        sample_grid[x][y] = 0;
        points.add(tmp1);
        active_list.add(tmp1);
        fill_sample_grid(sample_grid, tmp1);
        break;
      }
    }
        
    if (active_list.size() == active_list_size) active_list.remove(tmp);
  }
}
```
