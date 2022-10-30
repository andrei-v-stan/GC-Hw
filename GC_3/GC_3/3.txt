#include <iostream>
#include <GL/glut.h>
#include <stdlib.h>
#include <stdio.h>
#include <math.h>
#include <vector>
#include <math.h>
#include <utility>

#define dimension 300

using namespace std;

unsigned char prevKey;

class GrilaCarteziana{
public:
    GrilaCarteziana(int cols, int rows){
        this->rows = rows;
        this->cols = cols;

    }

    GrilaCarteziana(){
        this->cols = 10;
        this->rows = 10;

    }

    void writePixel(int row, int col){

        this->pixels.push_back(pair<int,int>(row,col));
        pair<int,int> coord = this->transformPixels(row,this->cols - col);

        int points = 100;
        float angle = 2.0f * 3.1416f / points;
        for(auto pixel:pixels) {
            glColor3f(0.4, 0.4, 0.4);
            glMatrixMode(GL_MODELVIEW);
            glPushMatrix();
            glLoadIdentity();
            glBegin(GL_POLYGON);
            double angle1 = 0.0;
            double radius = this->gridSpacing*0.4;
            glVertex2d(coord.first , coord.second);
            for (int i = 0; i < points; ++i)
            {
                glVertex2d(radius * cos(angle1) + coord.first, radius * sin(angle1) + coord.second);
                angle1 += angle;
            }
            glEnd();
            glPopMatrix();
        }
    }

    void drawCartGrid(){

        int colsNo = 0;
        for(int x = this->originx; x <= this->originx + this->cols*gridSpacing && colsNo<=this->cols ;x+=gridSpacing){
            glColor3f(0.4,0.4,0.4);
            glBegin(GL_LINES);
                glVertex2i(x,this->originy);
                glVertex2i(x,this->originy + gridSpacing*this->cols);
            glEnd();
            colsNo++;
        }

        int rowsNo = 0;
        for(int y = this->originy; y <= this->originy + this->rows*gridSpacing && rowsNo<= this->rows ;y+=gridSpacing){
            glColor3f(0.4,0.4,0.4);
            glBegin(GL_LINES);
                glVertex2i(this->originx,y);
                glVertex2i(this->originx + gridSpacing*this->rows,y);
            glEnd();
            rowsNo++;
        }

    }
    void drawSegmentLine3A(int x0, int y0, int xn, int yn,int thickness) {

        pair<int,int> lineCoordStart = this->transformPixels(x0,this->cols - y0);
        pair<int,int> lineCoordEnd = this->transformPixels(xn,this->cols - yn);

        glColor3f(1.0,0.0,0.0);
        glBegin(GL_LINES);
        glVertex2i(lineCoordStart.first,lineCoordStart.second);
        glVertex2i(lineCoordEnd.first,lineCoordEnd.second);
        glEnd();

        int dx = xn - x0, dy = yn - y0;
        int d = 2* dy - dx;
        int dE = 2 * dy;
        int dNE = 2 * (dy - dx);

        int x = x0,y = y0;

        for(int i = max(0, y - thickness); i<= min(y + thickness,this->rows); ++i) {
                this->writePixel(x,i);
        }
        this->writePixel(x,y);


        while(x < xn){

            if (d <= 0) {
                d += dE;
                x++;
            }
            else {
                d += dNE;
                x++;
                y++;
            }
            this->writePixel(x,y);

            for(int i = max(0, y - thickness); i<= min(y + thickness,this->rows); ++i) {
                this->writePixel(x,i);
            }
        }
    }

    void drawSegmentLine3B(int x0, int y0, int xn, int yn,int thickness) {

        pair<int,int> lineCoordStart = this->transformPixels(x0,this->cols - y0);
        pair<int,int> lineCoordEnd = this->transformPixels(xn,this->cols - yn);

        glColor3f(1.0,0.0,0.0);
        glBegin(GL_LINES);
        glVertex2i(lineCoordStart.first,lineCoordStart.second);
        glVertex2i(lineCoordEnd.first,lineCoordEnd.second);
        glEnd();

        int dx = xn - x0, dy = yn - y0;
        int d = 2* dy + dx;
        int dE = 2 * dy;
        int dSE = 2 * (dy + dx);

        int x = x0,y = y0;

        for(int i = max(0, y - thickness); i<= min(y + thickness,this->rows); ++i) {
                this->writePixel(x,i);
        }

        this->writePixel(x,y);

        while(x < xn){
            if (d > 0) {
                d += dE;
                x++;
            }
            else {
                d += dSE;
                x++;
                y--;
            }
            this->writePixel(x,y);

            for(int i = max(0, y - thickness); i<= min(y + thickness,this->rows); ++i) {
                this->writePixel(x,i);
            }
        }
    }


    pair<int,int> transformPixels(int i,int j){
        int xi = this->originx + this->gridSpacing * i;
        int yj = this->originy + this->gridSpacing * j;

        return pair<int,int>(xi,yj);
    }

    void setCols(int cols){
        this->cols=cols;
    }

    void setRows(int rows){
        this->rows=rows;
    }

    ~GrilaCarteziana(){}
private:
    int cols;
    int rows;
    int gridSpacing = min(glutGet(GLUT_WINDOW_WIDTH),glutGet(GLUT_WINDOW_HEIGHT))*0.05 ;
    double originx = glutGet(GLUT_WINDOW_WIDTH)/8;
    double originy = glutGet(GLUT_WINDOW_HEIGHT)/8;
    vector<pair<int,int>> pixels;

};


void Init(void) {
   glClearColor(1.0,1.0,1.0,1.0);
   glLineWidth(1);
   glPointSize(4);
   glPolygonMode(GL_FRONT, GL_LINE);
}

void Display(void) {
   GrilaCarteziana gc;
   gc.setCols(15);
   gc.setRows(15);
   glClear(GL_COLOR_BUFFER_BIT);
   gc.drawCartGrid();
   gc.drawSegmentLine3B(0,15,15,10,1);
   gc.drawSegmentLine3A(0,0,15,7,0);
   glFlush();
}

void Reshape(int w, int h) {
   glViewport(0, 0, (GLsizei) w, (GLsizei) h);
   glMatrixMode(GL_PROJECTION);
   glLoadIdentity();
   double windowWidth = glutGet(GLUT_WINDOW_WIDTH);
   double windowHeight = glutGet(GLUT_WINDOW_HEIGHT);
   glOrtho(0.0f, windowWidth, windowHeight, 0.0f, 0.0f, 1.0f);
}


void KeyboardFunc(unsigned char key, int x, int y) {
   prevKey = key;
   if (key == 27) // escape
      exit(0);
   glutPostRedisplay();
}

void MouseFunc(int button, int state, int x, int y) {
   printf("Call MouseFunc : ati %s butonul %s in pozitia %d %d\n",
      (state == GLUT_DOWN) ? "apasat" : "eliberat",
      (button == GLUT_LEFT_BUTTON) ?
      "stang" :
      ((button == GLUT_RIGHT_BUTTON) ? "drept": "mijlociu"),
      x, y);
}

int main(int argc, char** argv) {

   glutInit(&argc, argv);

   glutInitWindowSize(dimension, dimension);

   glutInitWindowPosition(100, 100);

   glutInitDisplayMode (GLUT_SINGLE | GLUT_RGB);

   glutCreateWindow (argv[0]);

   Init();

   glutReshapeFunc(Reshape);

   glutKeyboardFunc(KeyboardFunc);

   glutMouseFunc(MouseFunc);

   glutDisplayFunc(Display);

   glutMainLoop();

   return 0;
}
