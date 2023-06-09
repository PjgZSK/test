# Hello Triangle
## Quick Check
1. OpenGL quick check  
2. graphics pipeline  
3. shader  
4. the stages of graphics pipeline
5. vertex input  
6. vertex shader
7. fragment shader
8. shader program
9. link vertex attributes
10. vertex array object
11. draw triangle  

## OpenGL quick check  
* OpenGL is 3d and screen or windows is 2d  
* a large part of OpenGL's work is about transforming all 3d coordinates to 2d pixels than fit your screen  

## graphics pipeline  
* graphics pipeline of OpenGL manage the process of transforming 3d coordinates to 2d pixels  
* graphics pipeline :  
    1. transforming 3d coordinates to 2d coordinates  
    2. transforming 2d coordinates into actual colored pixels  
* the graphics pipeline can divided into serveral steps where each steps requires the output of previous step as its input  


## shader  
* all of the steps are highly specialized and can easily to be executed in parallel  
* because of parallel nature, graphics cards of today have thousands of small process cores to   
    quickly process your data within the graphics pipeline.  
* the processing cores run small programs on the GPU for each step of the pipeline  
* these small programs are called shaders  
* some of those shaders are configurable by the developer which allows us to write our own shaders to   
    replace the existing default shaders  
* shaders are written by OpenGL Shading Language(GLSL)  


## the stages of graphics pipeline
* abstract representation of all stages of graphics pipeline(note that italic sections represent sections where we can inject our own shaders)  : 
    input : vertex data  
    stages : *vertex shader* -> shape assemblely -> *geometry shader* -> rasterization -> *fragment shader* -> tests and blending  
* vertex data is a list of 3d coordinates than should form a triangle and is represented using vertex attributes that can contain any data  
    we like, but for simplicity's sake let's assume that each vertex consists of just a 3d position and some color value.  
* primitives  
    for OpenGL to know what to make of your collection of coordinates and color values OpenGL requires you to hint what kind of render types  
    you want to form with the data. those hints are called primitives and are gived to OpenGL while calling ang of the drawing commands.  
    some of these hints are GL_POINTS, GL_TRIANGLES and GL_LINE_STRIP.  
* vertex shader :  
    vertex shader takes as input a single vertex  
    the main purpose of the vertex shader is to transform 3d coordinates into different 3d coordinates and  
    allow us to do some basic processing on the vertex attributes.  
* geometry shader :  
    the output of the vertex shader stage is optionally passed to the geometry shader.  
    the geometry shader takes as input a collection of vertices that form a primitives and  
        has ability to generate other shapes by emitting new verteces to form new(or other) primitive(s).  
* primitive assemblely :  
    the primitive assemblely stage takes as input all the vertices form the vertex or geometry shader that form one  
        or more primitives and assembles all the points in the primitive shape given.  
* rasterization :  
    the output of the geometry shader is then passed on to the rasterization stage where is maps  
        the resulting primitives to the corresponding pixels on the final screen, result in fragments  
        for fragment shader to use.  
    before the fragment shaders run, *clipping* is performed.  
    clipping discard all fragments that are outside your view, increasing performance  
* fragment shader :  
    *a fragment in OpenGL is all the data required for OpenGL to render a single pixel*  
    the main purpose of fragment shader is to calculate the final color of a pixel and this  
        is usually the stage where all the advanced OpenGL effect occur.  
    usually the fragment shader contains data about 3d scene that it can use to calculate the final pixel color(like lights,  
        shadows, color of the light and so on.)  
* alpha test and blending :  
    when all corresponding color values have been determined, the final object will then pass through one more stage that  
        we call the alpha test and blending stage.  
    the stage check the corresponding depth(and stencil) value of fragment and use those to check if the resulting fragment  
        is in front of or behind other objects and should be discarded accordingly  
    the stage also check for alpha values(alpha values define the opacity of an object) and blends the objects accordingly.  
* other  
    tessellation stage  
    transform feedback loop  
* in modern OpenGL we are required to defined at least a vertex and fragment shader of our own(there is no default vertex/fragment  
    shaders on the GPU)  


## vertex input  
* normalized device coordinates(NDC)  
    NDC is in a small space where the x,y and z values vary from -1.0 to 1.0; all coordinates that fall outside this range  
        will be discarded/clipped and won't be visible on your screen.
    unlike usual screen coordinates, in NDC the positive y-axis in the up-direction and the (0,0) coordinates are at the  
        center of graph, instead of top-left.  
* glViewport  
    your NDC coordinates will then be transform to screen-space coordinates via the viewport transform using data you provided  
        with glViewport  
    the resulting screen-space coordinates are then transform to fragments as inputs to your fragment shader.  
    the function requires 4 coordinates for the left, bottom, right and top coordinates of your viewport rectangle. the coordinates  
        specified tell OpenGL how to map its NDC to *windows coordinates*(in the range as specified by the given coordinates)  
* define a triangle vertex data in NDC in a float array :  
    ```
    float vertices[] = {
        -0.5f, -0.5f, 0.0f,
        0.5f, -0.5f, 0.0f,
        0.0f, 0.5f, 0.0f
    };        
    ```
* send vertex data at input to the first process of the graphics pipeline : vertex shader  
    1. create memory on the GPU where we store the vertex data  
    2. configue how OpenGL should interpret the memory  
    3. specify how to send the data to the graphics card  
* *vertex buffer objects(VBO)*  
    vbo is OpenGL object that has a unique ID.  
    we manage GPU's vertex memory by vbo that can store a large number of vertex data in GPU's memory. the advantage of using   
        those buffer objects is that we can send large batches of data all at once to the graphics card, and keep it there if  
        there's enough memory left, without having to send data one vertex at once.  
    send data to graphics cards from cpu is relatively slow, so wherever we can we try to send as much data as possible at once.  
* glGenBuffers  
    we can generate one with a buffer ID using the *glGenBuffers* function :  
    ```
    unsigned int VBO;
    glGenBuffers(1, &VBO);
    ```
* glBindBuffer  
    OpenGL has many types of buffer objects and the buffer type of vertex buffer object is GL_ARRAY_BUFFER.  
    we can bind the newly created buffer to the GL_ARRAY_BUFFER target with glBindBuffer function :  
    ```
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    ```
    from this point on any buffer calls we make(on the GL_ARRAY_BUFFER target) will be used to configure the  
        currently bound buffers, which is `VBO`.  
* glBufferData  
    so we can make a call to the glBufferData that copies the previously vertex data into the buffer's memory :  
    ```
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    ```
    the function allocates memory and stores data within the initialized memory in the currently bound buffer object.  
    the fourth parameter specifies how we want graphics card to manager the given data. this can take 3 forms :  
        1. *GL_STREAM_DRAW* : the data is set only once and used by GPU at most a few times.  
        2. *GL_STATIC_DRAW* : the data is set only once and used by many times.  
        3. *GL_DYNAMIC_DRAW* : the data is changed a lot and used by many times.  
    the position data of the triangle does not change, is used a lot, and stay the same for every render call  
        so its usage type should best be *GL_STATIC_DRAW*.  


## vertex shader
* vector  
    In graphics programming we use the mathematical concept of a vector quite often, since it neatly  
        represents positions/directions in any space and has useful mathematical propertice.  
    A vector in GLSL has a maximum size of 4 and each of its values can be retrieved via *vec.x*,  
        *vec.y*, *vec.z* and *vec.w* respectively where each of them represents a coordinate in space.  
    Note that the *vec.w* component is not used as a position in space, but it used for something called  
        *perspective division*.
* vertex shader  
    ```
    #version 330 core
    layout (location = 0) in vec3 aPos;

    void main()
    {
        gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
    }
    ```
    Vertex shader begin with a declaration of its version. Since OpenGL 3.3 and higher the version  
        number of GLSL match the version of OpenGL(330 represent version 3.3). We also explicitly  
        mention that we're using core profile functionality.  
    Next we declare all the input vertex attributes in the vertex shader with the `in` keyword.  
        right now we only care about position data so we only need a single vertex attribute.  
    We also specifically set the location of the input variable via `layout (location = 0)`. 
    The gl_Position variable is predefined output variable and is a vec4 behind the scenes.
    The current vertex shader is probably the most simple vertex shader we can imagine because we  
        did no processing whatsoever on the input data and simply forwarded it to the shader's output.  
    In real applications the input data is usually not already in NDC so we first have to transform  
        the input data to coordinates that fall within OpenGL's visible region.
* compiling a shader  
    ```
    const char* vertexShaderSource = "#version 330 core\n"
        "layout (location = 0) in vec3 aPos;\n"
        "void main()\n"
        "{\n"
        "    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
        "}\0";
    ```
    We take the source code for vertex shader and store it in a const C string at the top of the code file.  
    ```
    unsigned int vertexShader;
    vertexShader = glCreateShader(GL_VERTEX_SHADER);
    ```
    In order for OpenGL to use the shader it has to dynamically compile it at run-time from its source code.  
    The first thing we need to do is create a shader object, again referenced by an ID.  
    So we store the vertex shader with an unsigned int and create the shader with glCreateShader.  
    We provide the type of shader we want to create as an argument to glCreateShader. the argument can  
        take one of the following values:  
            GL_COMPUTE_SHADER  
            GL_VERTEX_SHADER  
            GL_TESS_CONTROL_SHADER  
            GL_TESS_EVALUATION_SHADER  
            GL_GEOMETRY_SHADER  
            GL_FRAGMENT_SHADER  
    ```
    glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
    glCompileShader(vertexShader);
    ```
    Now we attach the shader source code to the shader object and compile shader.  
    The glShaderSource function :  
        `glShaderSource(GLuint shader, GLsizei count, const GLchar** string, const GLint* length)`  
        *shader* : shader unsigned reference  
        *count* : the number of elements in the *string* array  
        *string* : an array of points to strings containing the source code to be load in the shader  
        *length* : an array of string lengths
* check compilation results  
    Checking for compile-time error : 
    ```
    int success;
    char infoLog[512];
    glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);
    if (!success)
    {
        glGetShaderInfoLog(vertexShader, 512, NULL, infolog);
        std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
    }
    ```
    glGetShaderiv : query a shader for information given a object parameter.  
    glGetShaderInfoLog : return the information log for a shader object.  


## fragment shader
* RGBA  
    Color in computer graphics are represented as an array of 4 values : the red, green, blue and  
        alpha(opacity) component, commonly abbreviated to RGBA.  
    When defining a color in OpenGL or GLSL we set the strength of each component to a value between  
        0.0 and 1.0. Given those 3 color components we can generate over 16 million(256 bit in every conponent) different colors!  
* fragment shader  
    The fragment shader is all about calculate the color output of your pixels.  
    ```
    #version 330 core
    out vec4 FragColor;
    void main()
    {
        FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);
    }
    ```
    The fragment shader only requries one output variable and that is a vector of size 4 that defines the  
        final color output that we should calculate ourselves.  
    We can declare the output value with the *out* keyword, that we here promptly named *FragColor*.  
    Next we assign a vec4 to the color output as an orange color with an alpha value of 1.0(1.0 being  
        completely opaque).  
* compiling fragment shader  
    The process of compiling a fragment shader is similar to the vertex shader.  
    ```
    unsigned int fragmentShader;
    fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
    glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
    glCompileShader(fragmentShader);
    ```
    Both the shaders are now compiled and the only thing left to do is link both shader objects into  
        a *shader program* that we can use for rendering. Make sure to check compile error here as well.  


## shader program
* shader program  
    A shader program is the final version of multiple shader combined. To use the recently compiled  
        shaders we have to *link* them to a shader program object and then activate this shader  
        program when rendering objects. The activated shader program's shaders will be used when  
        we issue render calls.
    When linking the shaders into a program it links the outputs of each shader to the inputs of  
        the next shader. This is also where you'll get linking errors if your outputs and inputs  
        do not match.  
    ```
    unsigned int shaderProgram;
    shaderProgram = glCreateProgram();
    glAttachShader(shaderProgram, vertexShader);
    glAttachShader(shaderProgram, fragmentShader);
    glLinkProgram(shaderProgram);
    ```
    The glCreateProgram functions creates a program and return the ID reference to newly created   
        program object.  
    Now we need to attach the previously compiled shaders to the program object and then link  
        them with glLinkProgram.  
    ```
    glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
    if (!success)
    {
        glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog);
        ...
    }
    ```
    Just check shader compilation we can also check if linking a shader program failed and  
        retrieve the corresponding log.  
    ```
    glDeleteShader(vertexShader);
    glDeleteShader(fragmentShader);
    ```
    Don't forget to delete the shader objects once we've linked them into the program object.  
    We no longer need them anymore.  
* We are almost there, but not quite yet.  
    OpenGL does not yet know how it should interpret the vertex data in the memory and how  
        it should connect the vertex data to the vertex shader's attributes.


## link vertex attributes
* linking vertex attributes  
    The vertex shader allow us to specify any input we want in the form of vertex attributes  
        and while this allows for great flexibility, it does mean we have to manually specify  
        what part of our input data goes to which vertex attributes in the vertex shader.  
    This means we have to specify how OpenGL should interpret the vertex data before rendering.  
* glVertexAttribPointer  
    ```
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
    glEnableVertexAttribArray(0);
    ```
    ```
    glVertexAttribPointer(GLuint index, GLint size, GLenum type, GLboolean normalized, GLsizei
        stride, const GLvoid * pointer)
    ```
    *index* : specify which vertex attribute we want to configue. Remember that we specified  
        the location of position vertex attribute in the vertex shader with *layout (location = 0)*  
        This sets the location of the vertex attribute to 0 and since we want to pass data to  
        this vertex attribute, we pass in 0.
    *size* : specify the size of vertex attribute. The vertex attribute is a vec3 so it is composed  
        of 3 values.  
    *type* : specify the type of the data which is GL_FLOAT.  
    *normalized* : specify if we want the data to be normalized.  
    *stride* : tell us the space between consecutive vertex attribute.  
    *pointer* : offset of where the position data begins in the buffer.  
    Each vertex attribute takes its data from memory managed by VBO and which VBO it takes its data  
        form is determined by the VBO currently bound to GL_ARRAY_BUFFER when calling glVertexAttribPointer.  
        Since the previously defined VBO is still bound before calling glVertexAttribPointer vertex  
        attribute 0 is now associated with its vertex data.  
* glEnableVertexAttribArray  
    Now that we specified how OpenGL should interpret the vertex data we should also enable the vertex  
        attribute with glEnableVertexAttribArray giving the vertex attribute location as its argument.  
    Vertex attribute are disabled by default.  
    From that points on we have everything set up :  
        1. we initialized the vertex data in a buffer using a vertex buffer object.  
        2. set up a vertex and fragment shader.  
        3. tell OpenGL how to link the vertex data to the vertex shader's vertex attributes.  
    ```
    //0. copy our vertices array in a buffer for OpenGL to use
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    //1. then set the vertex attributes pointers
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
    glEnableVertexAttribArray(0);
    //2. use our shader program when we want to render an object
    glUseProgram(shaderProgram);
    //3. now draw the object
    someOpenGLFunctionThatDrawOurTriangle();
    ```
    We have to repeat this process every time we want to draw an object.  
    What if there was some way we could store all these state configurations into a object and  
        simply bind this object to restore its state ?  


## vertex array object
* vertex array object  
    A vertex array object (also know as VAO) can be bound just like a vertex buffer object and  
        any subsequent vertex attribute calls from that point on will be stored inside the VAO.  
    This has the advantage that when configuring vertex attribute pointers you only have to make  
        those calls once and whenever we want to draw the objects, we can just bind the corresponding  
        VAO.  
    *Core OpenGL requires that we use VAO so it knows what to do with our vertex inputs. If we  
        fail to bind VAO, OpenGL will most likely refuse to draw anything.*  
    A vertex array object stores following :  
        1. Calls to glEnableVertexAttribArray or glDisableVertexAttribArray.  
        2. Vertex attribute configurations via glVertexAttribPointer.  
        3. Vertex buffer objects associated with vertex attributes by calls to glVertexAttribPointer.  
* use VAO  
    The process of generate VAO is similar with VBO :  
    ```
    unsigned int VAO;
    glGenVertexArrays(1, &VAO);
    ```
    To use a VAO all you have to do is bind the VAO using glBindVertexArray.  
    From that point on we shoud bind/configure the corresponding VBO and attribute pointer and then  
        unbind the VAO for later use.  
    As soon as we want to draw an object, we simply bind the VAO with the preferred setttings before  
        drawing the object and that is it.  
    ```
    // ..:: Initialization code (done once (unless your object frequently changes)) ::..
    // 1. bind vertex array object
    glBindVertexArray(VAO);
    // 2. copy our vertices array in a buffer for OpenGL to use  
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    // 3. then set our vertex attribute pointers
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
    glEnableVertexAttribArray(0);
    
    [...]

    // ..:: Drawing code (in render loop) ::..
    // 4. draw the object
    glUseProgram(shaderProgram);
    glBindVertexArray(VAO);
    someOpenGLFunctionThatDrawOurTriangle();
    ```
    A VAO stores our vertex attribute configuration and which VBO use. Usually when  
        you have multiple objects you want to draw, you first generate/configure all  
        the VAOs(and thus the requires VBO and attribute pointers) and stores those  
        for later use.  
    The moment we want to draw one of our objects, we take the corresponding VAO,  
        bind it, then draw the object and unbind VAO again.  


## draw triangle  
* draw triangle  
    To draw our objects of choice, OpenGL provides us with the glDrawArrays function that  
        draw primitives using the currently active shader, the previously defined vertex  
        attribute configuration and with the VBO's vertex data(indirectly bound via VAO).  
    ```
    glUseProgram(shaderProgram);
    glBindVertexArray(VAO);
    glDrawArrays(GL_TRIANGLE, 0, 3);
    ```
    `glDrawArrays(GLenum mode, GLint first, GLsizei count)`  
    *mode* : Specifies the kind of primitive to render, can take the follow values:  
        GL_POINTS  
        GL_LINE_STRIP  
        GL_LINE_LOOP  
        GL_LINES  
        GL_TRIANGLE_STRIP  
        GL_TRIANGLE_FAN  
        GL_TRIANGLES  
        GL_QUAD_STRIP  
        GL_QUADS  
        GL_POLYGON  
    *first* : Specifies the staring index of the vertex array we like to draw  
    *count* : Specifies how many vertices we want to draw  

## Element Buffer Objects  
1. There is one last thing we'd like to discuss when rendering vertices and that  
is element buffer objects abbreviated to EBO.  
2. Suppose we want to draw a rectangle instead of a triangle. We can draw a rectangle  
using two triangles(OpenGL mainly works with triangles). This will generate the following  
set of vertices :  
```
float vertices[] = {
    // first triangle
    0.5f, 0.5f, 0.0f, // top right
    0.5f, -0.5f, 0.0f, // bottom right
    -0.5f, 0.5f, 0.0f, // top left
    // second triangle
    0.5f, -0.5f, 0.0f, // bottom right
    -0.5f, -0.5f, 0.0f, // bottom left
    -0.5f, 0.5f, 0.0f // top left
```
3. As you can see, there is some overlap on the vertices specified . We specify bottom right   
and top right twice. This is an overhead of 50% since the same rectangle could also be specified  
by 4 vertices, instead of 6. This will get worse as soon as we have more complex models that  
have over 1000s of triangles where there will be large chunks that overlap. What would be  
a better solution is to store only the unique vertices and then at which we want to draw  
these vertices in. In that case we only need store 4 vertices for the rectangle, and then  
just specify at which order we'd like to draw them.  
4. Thankfully, element buffer objects work exactly like that. An EBO is a buffer, just like  
a vertex buffer object, that stores the indices that OpenGL uses to decide what vertices   
to draw. This is so called indexed drawing is exactly the solution to our problem.   
5. To get started we first have to specify the (unique) vertices and the indices to draw  
them as a rectangle.  
```
float vertices[] = {
    0.5f, 0.5f, 0.0f, // top right
    0.5f, -0.5f, 0.0f, // bottom right 
    -0.5f, -0.5f, 0.0f, // bottom left  
    -0.5f, 0.5f, 0.0f // top left
```
You can see that, when using indices, we only need 4 vertices instead of 6.  
6. Next we need to creat the element buffer object :  
```
unsigned int EBO;
glGenBuffers(1, &EBO);
```
Similar to the VBO we bind the EBO and copy the indices into the buffer with glBufferData.  
Also, just like the VBO we want to place those calls between a bind and an unbind call,  
although this time we specify GL_ELEMENT_ARRAY_BUFFER as the buffer typy.  
```
glBindBuffer(EBO, GL_ELEMENT_ARRAY_BUFFER);
glBufferData(EBO, sizeof(indices), indices, GL_STATIC_DRAW);
```
Note that we're giving the GL_ELEMENT_ARRAY_BUFFER as buffer target. The last thing left  
to do is replace the glDrawArrays call with glDrawElements to indicate we want to  
render the triangles from an index buffer. When using glDrawElements we're going to draw  
using indices provided in the element buffer object currently bound :  
```
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
```
The first argument specifies the mode we want to draw in, similar to glDrawArrays.  
The second argument is the count or number of elements we'd like to draw. We specify  
6 indices so we want to draw 6 vertices in total.  
The third argument is the type of the indices which is of type GL_UNSIGNED_INT.  
The last argument allows us to specify an offset in the EBO(or pass in an index  
array, but that is when you're not using element buffer object), but we're just  
going to leave this at 0.  
7. The glDrawElements function takes its indices from the EBO currently bound to  
the GL_ELEMENT_ARRAY_BUFFER target. This means we have to bind the corresponding  
EBO each time we want to render an object with indices which again is a bit  
cumbersome. It just so happens that a vertex array object also keep track  
of element buffer object bindings. The last element buffer object that gets  
bound while a VAO is bound, is stored as the VAO's element buffer object. Binding  
to a VAO then also automatically binds that EBO.  
8. *A VAO stores the glBindBuffer when the target is GL_ELEMENT_ARRAY_BUFFER.  
This also means it stores unbind calls so make sure you don't unbind the  
element array buffer before unbinding your VAO, otherwise it doesn't have  
an EBO configured.*  
9. The resulting initialization and drawing code now looks something like this :  
```
// ..:: Initialization code ::..
// 1. bind vertex array object
glBindBuffer(VAO);
// 2. copy our vertices array in vertex buffer for OpenGL to use
glBindBuffer(GL_ARRAY_BUFFER, VAO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
// 3. copy our index array in a element buffer for OpenGL to use
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);
// 4. then set the vertex attributes pointers
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);

[...]

// ..:: drawing code (in render loop) ::..
glUseProgram(shaderProgram);
glBindVertexArray(VAO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
glBindVertexArray(0);
```

## Additional resources
1. [open.gl/drawing](https://open.gl/drawing) : Alexander Overvoorde's take on rendering the first triangle.  
2. [learnopengl.com/In-Practice/Debugging](https://learnopengl.com/In-Practice/Debugging) :  
There are a lot of steps involved in this chapter; If you're stuck it may be worthwhile to read a bit  
on debugging in OpenGL(up until the debug output section).

## Exercises
1. Try to draw 2 triangles next to each other using glDrawArrays by adding more vertices to your data.  
2. Now create two same triangles using two different VAOs and VBOs for their data.  
3. Create two shader program where the second use a different fragment shader that outputs the color  
yellow; draw both triangles again where one outputs the color yellow.  


## Question
* how graphics cards work with parallel graphics pipeline ?

link : https://learnOpenGL.com/Getting-started/Hello-Triangle
