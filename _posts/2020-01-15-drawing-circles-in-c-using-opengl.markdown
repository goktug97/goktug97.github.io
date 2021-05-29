---
layout: post
title:  "Drawing Circles in C using Modern OpenGL"
date:   2020-01-15 19:43:14 +0300
categories: tutorial
tags: [opengl, c]
video: /assets/images/circles.mp4
---

Recently, I wrote [a library for the algorithm dynamic window approach in
C](https://github.com/goktug97/DynamicWindowApproach). I wanted to write
examples to show how to use the library. So, I've decided to create an
interactive demo where the robot follows the user's mouse. The user can
draw obstacles and the robot will avoid them using the algorithm. The
library has python bindings so I created the said demo using OpenCV GUI but
the actual library is written in C and can be used with C, so I also needed
to write a C example. I decided to use SDL and OpenGL. It seems that the
OpenGL has legacy and modern versions and I used legacy features
unknowingly. After creating the working examples, I wanted to create a web
application version to put into my site. I found Emscripten library which
can compile C applications to JavaScript but the problem is my C example is
written in legacy OpenGL and I couldn't make it to work with it. So I had
to write it from scratch in modern OpenGL. After a lot of frustrations, I
managed to make it work but the hardest part for me to figure out how to
draw a lot of circles with high performance. This is my first time using
OpenGL so if there is a better way to do this, I would like to hear it.

[DWA Demo](/dwa)

### SDL Window
The Emscripten version is written with SDL because SDL2 required
additional work but I am going to use SDL2 in this post. If you are
interested in the Emscripten version, you can find it
[here](https://github.com/goktug97/DWAGL).

Let's create a SDL window where we can draw something. The window also
catches mouse events so that we can program it to act as a paint
brush.

```c
#include <SDL2/SDL.h>
#include <GLES3/gl3.h>

#define RESOLUTION_WIDTH 640
#define RESOLUTION_HEIGHT 480

int main() {
  if (SDL_Init(SDL_INIT_EVERYTHING) == -1) return 1;

  // SDL Window
  SDL_Window *window =
    SDL_CreateWindow("float", 0, 0, RESOLUTION_WIDTH, RESOLUTION_HEIGHT, 
                     SDL_WINDOW_OPENGL|SDL_WINDOW_RESIZABLE);
  SDL_GLContext glcontext = SDL_GL_CreateContext(window);

  int running = 1;
  int drawing = 0;

  while (running) {
    SDL_Event sdlEvent;
    glClear(GL_COLOR_BUFFER_BIT);
    glClearColor(0.0, 0.0, 0.0, 1);

    while (SDL_PollEvent(&sdlEvent) > 0) {
      switch (sdlEvent.type) {
      case SDL_QUIT:
        running = 0;
        break;
      case SDL_KEYDOWN:
        switch (sdlEvent.key.keysym.sym) {
        case SDLK_ESCAPE:
          running = 0;
          break;
        case SDLK_r:
          // Reset
          break;
        }
        break;
      case SDL_MOUSEBUTTONDOWN:
        if (sdlEvent.button.which == 0)
          drawing = 1;
        break;
      case SDL_MOUSEBUTTONUP:
        drawing = 0;
        break;
      case SDL_MOUSEMOTION:
        if (drawing) {
          // Fill the point buffer
        }
        break;
      }
    }
    SDL_GL_SwapWindow(window);
  }
  SDL_GL_DeleteContext(glcontext);
  SDL_Quit();
  return 0;
}
```

Build It
```bash
gcc circles.c -lSDL2 -lGLESv2 -o circles
```

Now we have a window to work with.

<div style="display:flex">
     <div style="flex:1">
         <center>
         <img class="b-lazy" src=data:image/png;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw== data-src="/assets/images/sdl_window.jpg" style="display: block; margin: auto; width: 100%;"/>
         </center>
     </div>
</div>

### OpenGL

I follewed [this](https://open.gl/drawing) tutorial to draw basic
shapes. After being able to draw a circle, I tried to draw bunch of
them in a loop but it was too slow. I found instancing technique that
enables users to draw the same object multiple times in a one draw
call. I followed
[this](https://learnopengl.com/Advanced-OpenGL/Instancing) tutorial to
learn about it.

Let's start with vertices for the circle.

<div style="display:flex">
     <div style="flex:1;">
         <video class="b-lazy" data-src="/assets/images/circle_cos_sin.mp4" type="video/mp4" autoplay muted playsinline loop style="display: block; margin: auto;" ></video>
         <div class="text-block">
         <center>
         <p>
         <a href="https://commons.wikimedia.org/wiki/File:Circle_cos_sin.gif">Source</a> 
         </p>
         </center>
         </div>
     </div>
</div>


I am going to use triangle fan method where you draw bunch of
triangles around a point to create a circle. The first vertex is
center of the fan. Each vertex connects with the vertex before it and
the first vertex to create a triangle. The location and the scale of the
circle will be changed from the vertex shader. Hence, we only need to
create unit direction vectors to find vertex points.

```c
#include <math.h>
#define N_TRIANGLES 10

int main() {
  ...

  int nVertices = N_TRIANGLES + 2;
  float vertices[nVertices*2];
  vertices[0] = 0.0f; // x
  vertices[1] = 0.0f; // y
  int circleIdx = 1;
  for (int i = 2; i < nVertices*2; i+=2) {
    vertices[i] = (cos(circleIdx * 2.0 * M_PI / N_TRIANGLES)); // x
    vertices[i+1] = (sin(circleIdx * 2.0 * M_PI / N_TRIANGLES)); // y
    circleIdx++;
  }

  ...
  
  while (running) {
    ...
  }
  return 0;
}
```

#### Buffers
```c
int main() {
  ...
  
  // float vertices[nVertices*2];

  // Buffers
  GLuint vao;
  glGenVertexArrays(1, &vao);
  glBindVertexArray(vao);

  GLuint vbo;
  glGenBuffers(1, &vbo);
  glBindBuffer(GL_ARRAY_BUFFER, vbo);
  glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

  ...
  
  while (running) {
    ...
  }
  SDL_GL_DeleteContext(glcontext);
  SDL_Quit();
  glDeleteBuffers(1, &vbo);
  glDeleteVertexArrays(1, &vao);
  return 0;
}
```

#### Shaders
```c
#include <string.h>
#include "cglm/cglm.h"

#define N_POINTS 1000

typedef struct {
  float x;
  float y;
} Point;

int main() {
  int currentIdx = 0;
  Point pointCloud[N_POINTS];

  mat4 translations[N_POINTS]; // For instancing
  
  // float vertices[nVertices*2];
  // GLuint vao;
  // GLuint vbo;

  const char* vertexFormat =
    "#version 300 es\n"
    "in vec2 position;\n"
    "uniform  mat4 trans[%d];\n"
    "void main() {\n"
    "mat4 t = trans[gl_InstanceID];\n"
    "gl_Position = t * vec4(position, 0.0, 1.0);\n"
    "}\0";
  char* vertexSource = malloc(sizeof(char) * strlen(vertexFormat) + 10);
  sprintf(vertexSource, vertexFormat, N_POINTS);

  const char* fragmentSource =
    "#version 300 es\n"
    "precision mediump float;\n"
    "uniform vec3 color;\n"
    "out vec4 outColor;\n"
    "void main() {\n"
      "outColor = vec4(color, 1.0);\n"
    "}\0";

  // Compile Shaders
  GLuint vertexShader = glCreateShader(GL_VERTEX_SHADER);
  glShaderSource(vertexShader, 1, (const GLchar * const*)&vertexSource, NULL);
  glCompileShader(vertexShader);

  GLuint fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
  glShaderSource(fragmentShader, 1, &fragmentSource, NULL);
  glCompileShader(fragmentShader);

  // Link Shaders
  GLuint shaderProgram = glCreateProgram();
  glAttachShader(shaderProgram, vertexShader);
  glAttachShader(shaderProgram, fragmentShader);
  glLinkProgram(shaderProgram);
  glUseProgram(shaderProgram);

  // Uniforms
  GLint uniColor = glGetUniformLocation(shaderProgram, "color");
  GLint uniTrans = glGetUniformLocation(shaderProgram, "trans");

  GLint posAttrib = glGetAttribLocation(shaderProgram, "position");
  glEnableVertexAttribArray(posAttrib);
  glVertexAttribPointer(posAttrib, 2, GL_FLOAT, GL_FALSE, 0, 0);
  
  while (running) {
    ...
  }

  SDL_GL_DeleteContext(glcontext);
  SDL_Quit();
  free(vertexSource);
  glDeleteProgram(shaderProgram);
  glDeleteShader(fragmentShader);
  glDeleteShader(vertexShader);
  glDeleteBuffers(1, &vbo);
  glDeleteVertexArrays(1, &vao);
  return 0;
}
```

#### Drawing

```c
int main() {
  ...

  while (running) {
    SDL_Event sdlEvent;
    glClear(GL_COLOR_BUFFER_BIT);
    glClearColor(0.0, 0.0, 0.0, 1);
    while (SDL_PollEvent(&sdlEvent) > 0) {
      switch (sdlEvent.type) {
      case SDL_QUIT:
        running = 0;
        break;
      case SDL_KEYDOWN:
        switch (sdlEvent.key.keysym.sym) {
        case SDLK_ESCAPE:
          running = 0;
          break;
        case SDLK_r:
          currentIdx = 0;
          break;
        }
        break;
      case SDL_MOUSEBUTTONDOWN:
        if (sdlEvent.button.which == 0)
          drawing = 1;
        break;
      case SDL_MOUSEBUTTONUP:
        drawing = 0;
        break;
      case SDL_MOUSEMOTION:
        // Fill pointCloud buffer
        if (drawing) {
          if (currentIdx < N_POINTS) {
            pointCloud[currentIdx].x =
              ((float)sdlEvent.button.x/(float)RESOLUTION_WIDTH-0.5) * 2.0;
            pointCloud[currentIdx].y =
              (-(float)sdlEvent.button.y/(float)RESOLUTION_HEIGHT+0.5) * 2.0;
            currentIdx++;
          } else {
            printf("Number of Points Limit: %d\n", N_POINTS);
          }
        }
        break;
      }
    }
    if (currentIdx) {
      // Fill translations
      for (size_t i = 0; i < currentIdx; i++) {
        // Location
        glm_mat4_identity(translations[i]);
        glm_translate(translations[i], (vec3){
            pointCloud[i].x,
              pointCloud[i].y,
              0.0f});

        glm_scale(translations[i], (vec3){0.01f, 0.01f, 1.0f}); // Size
        glm_rotate(translations[i], 0, (vec3){0.0f, 0.0f, 1.0f}); // Rotation 
        glUniform3f(uniColor, 0.0f, 1.0f, 1.0f); // Color
      }
      // Send to uniform
      glUniformMatrix4fv(uniTrans, currentIdx, GL_FALSE, translations[0][0]);
      // Draw
      glDrawArraysInstanced(GL_TRIANGLE_FAN, 0, nVertices, currentIdx);
    }
    SDL_GL_SwapWindow(window);
  }
  SDL_GL_DeleteContext(glcontext);
  SDL_Quit();
  free(vertexSource);
  glDeleteProgram(shaderProgram);
  glDeleteShader(fragmentShader);
  glDeleteShader(vertexShader);
  glDeleteBuffers(1, &vbo);
  glDeleteVertexArrays(1, &vao);
  return 0;
}
```

Build it
```bash
gcc main.c -lSDL2 -lGLESv2 -lm -o circles -I ./cglm/include/
```


[CGLM Library](https://github.com/recp/cglm)

### Result

<video class="b-lazy" data-src="/assets/images/circles.mp4" type="video/mp4" autoplay muted playsinline loop style="display: block; margin: auto; width=100%" ></video>

### Full Code

[https://github.com/goktug97/OpenGLCircles](https://github.com/goktug97/OpenGLCircles)
