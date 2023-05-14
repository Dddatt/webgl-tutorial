## <div align='center'>About WebGL</div>

<br>WebGL is a rasterization API used for rendering graphics directly within a webpage, based off of OpenGL. WebGL uses JavaScript and offers many features, being widely used for 3D programs within HTML websites.<br><br>WebGL has two versions, the standard WebGL, and WebGL2. WebGL2 is a more modern version that offers many more features compared to WebGL1. WebGL2 is backwards compatible with WebGL1.<br><br>WebGL utilizes your computer's Graphics Processing Unit to rasterize pixels at lightning speeds. It takes information you provide and does math to color pixels.<br><br>Most modern browsers and machines support WebGL1. WebGL2's support may be less, but still quite high. It is supported by major browsers, but some require the experimental version.<br><br>WebGL programs run using a special shader language similar to C++, called Graphics Library Shader Language(GLSL). GLSL is a type-strict language used to write the vertex and fragment shaders of a WebGL program.<br><br>Most of the WebGL API is boilerplate code, and the rest is for rendering.

## <div align='center'>Rendering a Triangle</div>

<br>To get started with WebGL, you'll first need to prepare an HTML program. In this tutorial, we'll be strictly using WebGL2.<br><br>Create a canvas element that'll be used for rendering the graphics.

```
<canvas id='myCanvas' width='500' height='500'></canvas>
```
Then, create a script that gets a WebGL context. You can use _"webgl"_, _"webgl2"_, or _"experimental-webgl"_. Inside the script...
```
//fetches the canvas element
let canvas=document.getElementById('myCanvas')

//the webgl api
let gl=canvas.getContext('webgl2')

//if webgl2 is not supported
if(!gl){
    
    alert('Your browser does not support WebGL2!')
    return
}
```
Now that you've got the WebGL context, it's time to start utilizing it, starting off with a basic background color and setting the viewport.

```
//color values are in RGBA format. each value is in the range of 0-1
gl.clearColor(0.1,0,0,1)

//clears the color buffer bits, showing the background color
gl.clear(gl.COLOR_BUFFER_BIT)


//sets the viewport. the first 2 params are the x and y offset values of the viewport from the lower left corner. the last 2 are the width and height
//in webgl, the up direction are positive y values
gl.viewport(0,0,canvas.width,canvas.height)
```
The canvas should now be filled with a dark maroon color. Now, it's time for the painful process of rendering a triangle.<br><br>We need to make a WebGLProgram object. Here's a helpful function:

```
//a function that returns a webglprogram object given the shader text
function createProgram(vshText,fshText){
    
    //these lines create and compile the shaders
    vsh=gl.createShader(gl.VERTEX_SHADER)
    fsh=gl.createShader(gl.FRAGMENT_SHADER)
    gl.shaderSource(vsh,vshText)
    gl.shaderSource(fsh,fshText)
    gl.compileShader(vsh)
    gl.compileShader(fsh)
    
    //these lines create the program and attaches the shaders
    let p=gl.createProgram()
    gl.attachShader(p,vsh)
    gl.attachShader(p,fsh)
    gl.linkProgram(p)
    
    return p
}
```
Now, we use the previous function to create a WebGLProgram. We need to create the GLSL shaders first. The shaders are defined as strings and passed into the function. We'll use template literals to make the code neat while allowing for line breaks.<br><br>In WebGL2, GLSL shaders must have _"#version 300 es"_ at the very start of the string. The vertex shader does math using information passed in to determine the position of vertices. The fragment shader determines the color of fragments(aka. pixels).

```
let vertexShaderCode=`#version 300 es
    
    //this line tells the shader what the presicion is. This is need in both the shaders. 'lowp' means low, 'mediump' means medium, and 'highp' is high.
    precision mediump float;
    
    //this is the postition of the vertices, and will be passed in from the buffers later
    in vec2 vertPos;
    
    void main(){
        
        //gl_Position is a vec4. Your vertex shader needs to do math to find out where the vertex is. Because this is a 2D program, just put the vertex position at where it is passed in, without any changes.
        
        //GLSL is very cool when dealing with vectors. The vertPos is a vec2, but it can be inserted into a vec4 with 2 other nums, adding up to 4 components.
        gl_Position=vec4(vertPos,0,1);
    }
`

let fragmentShaderCode=`#version 300 es
    
    precision mediump float;
    
    //In WebGL1, you set gl_FragColor, just like gl_Position, but in WebGL2, you use an 'out' vec4 statement, with the name of the output. The name can't be 'gl_FragColor'
    out vec4 fragColor;
    
    void main(){
        
        //We set the color here. colors are in RGBA in the range of 0-1
        //this makes all the computed pixels green
        fragColor=vec4(0,1,0,1);
    }
`


```
Now, we create the WebGLProgram using the helpful function:

```
let program=createProgram(vertexShaderCode,fragmentShaderCode)

//now we tell webgl to use the program we made
gl.useProgram(program)
```
Our WebGLProgram is now initalized. When you call _gl.useProgram()_, it sets the current program being used as whatever program you put in. Graphics will be rendered with that program.<br><br>Now, it's time to create the mesh and buffers. Buffers are like arrays of numbers that can only be accessed by the GPU. A "mesh" is just data that describes the geometry being drawn.<br><br>Here, we define a mesh and create a buffer:

```
//the vertices of the triangle. the coords are in the range of -1 to 1, for both the axes. canvas width and height don't matter
//the vertex positions are stored together. in this case, the format is x1,y1,x2,y2,x3,y3
let verts=[
    
    //in the middle, upwards
    0,0.5,
    
    //on the left, downwards
    -0.5,-0.5,
    
    //on the right, downwards
    0.5,-0.5,
]


//creates the buffer
let buffer=gl.createBuffer()

//tells webgl that we are performing stuff on this specific buffer
//gl.ARRAY_BUFFER indicates that this buffer is for vertex data
gl.bindBuffer(gl.ARRAY_BUFFER,buffer)

//sets data in the buffer using the mesh. the inputted data must be a Float32Aray 
gl.bufferData(gl.ARRAY_BUFFER,new Float32Array(verts),gl.STATIC_DRAW)
```
<br>The buffer is ready. Now, we pass them into the GLSL shaders as attributes. The GLSL vertex shader defines _"vertPos"_ as a vec2(2 numbers, forming a 2D vector). The _"in"_ keyword before it indicates that it's an attribute, which is data(a number or groups of numbers(as in vec2)) that is defined and differs for each vertex. Attributes only exist in the vertex shader.<br><br>Here, we get the location of the attribute(from the WebGLProgram) and use it to tell WebGL how the vertex array's data is formatted(in this case, each vertex has an x and y value combined as a vec2 attribute).

```
//gets the location of the attribute from the WebGLProgram
//the name(here it's "vertPos") must match what's defined in the vertex shader(the shader states "in vec2 vertPos;")
let vertPosLocation=gl.getAttribLocation(program,'vertPos')

//enable the vertex attribute
gl.enableVertexAttribArray(vertPosLocation)

//values in attribute. 2 values(x and y coords) are used in the "vertPos" attribute
let via=2

//bytes per vertex. the total amount of values per a vertex(here it's 2) multiplied by 4(which is the amount of bytes in a float32)
let bpv=8

//current attribute offset bytes. here, the "vertPos" attribute is the first attribute, so 0 values before it. the amount of bytes is the value multiplied by 4(which is the amount of bytes in a float32)
let caob=0 

//tells webgl how to get values out of the supplied buffer
gl.vertexAttribPointer(vertPosLocation,via,gl.FLOAT,gl.FALSE,bpv,caob)
```
Finally, we render the elements will a simple draw call! A draw call is just a call to _gl.drawArrays()_ or _gl.drawElements()_. Draw calls are quite expensive compared to the rest of the functions. You likely want less than 50 to 350 draw calls per frame for a good frame rate.

```
//1st param: type of primitive being drawn
//2nd param: starting vertex(often kept as 0)
//3rd param: amount of vertices
gl.drawArrays(gl.TRIANGLES,0,3)
```
You should now see a green triangle! It may seem like a lot, but the code won't expand much for more complicated programs.<br><br>A draw call uses the currently bound buffer and attributes. It's good practice to bind the buffer, make the _gl.vertexAttribPointer()_ calls, and then draw.

## <div align='center'>Varyings & More Attributes</div>

<br>In vertices, you can store much more information other than the vertex's position! Some common examples are vertex colors and normals. In this section, you'll learn how to create a multi-colored triangle.<br><br>We can use varyings to help accomplish this. A varying is a value that is set in the vertex shader and passed into the fragment shader. The value of the varying is set at each vertex, so what happens when a fragment(aka pixel) is between the vertices? The value is interpolated(or mixed) across the vertices, based on where the fragment is.<br><br>Starting off with the code for a basic triangle, we need to add colors to the vertices. Begin by adding a color attribute and a color varying to the vertex shader. The vertex shader now becomes...

```
let vertexShaderCode=`#version 300 es
    
    precision mediump float;
    
    in vec2 vertPos;

    //to pass in an RGB color as an attribute, we'll use a vec3 to store the values
    in vec3 vertColor;

    //in webgl1, use the varying keyword. in webgl2, we use the out keyword while inside the vertex shader
    out vec3 pixelColor;
    
    void main(){
        
        //here, we set the varying so it can be passed into the fragment shader
        pixelColor=vertColor;
        
        gl_Position=vec4(vertPos,0,1);
    }
`
```
<br>In the fragment shader, we set the color based on the varying value passed in from the vertex shader.

```
let fragmentShaderCode=`#version 300 es
    
    precision mediump float;
    
    //in webgl1, also use the varying keyword. in webgl2, we use the in keyword while inside the fragment shader
    in vec3 pixelColor;
    
    out vec4 fragColor;
    
    void main(){
        
        //We set the color as the value of the varying
        //interpolation is done automatically
        //notice that the varying("pixelColor") is a vec3, but "fragColor" needs to be a vec4. we combine the RGB value with an alpha value of 1(fully visible)
        fragColor=vec4(pixelColor,1);
    }
`
```
Now, we just need to add the color values to our mesh and create another attribute.<br><br>The updated mesh, with added color values takes the format of x1,y1,r1,g1,b1,x2,y2,r2,g2,b2,x3,y3,r3,g3,b3...

```
let verts=[
    
    //in the middle, upwards, red color
    0,0.5,  1,0,0,
    
    //on the left, downwards, green color
    -0.5,-0.5,  0,1,0,
    
    //on the right, downwards, blue color
    0.5,-0.5,  0,0,1
]
```
We don't need to change the way the buffer is created, but we need to update the way values are read from the buffer(the attributes). Create a new attribute for the color values, similar to the way the _"vertPos"_ attribute was created.

```
let vertColorLocation=gl.getAttribLocation(program,'vertColor')
gl.enableVertexAttribArray(vertColorLocation)
```
Now, we have to specify the new way values are read out of the buffer:

```
//bytes per vertex. the total amount of values per a vertex(now it's 5(x,y,r,g,b)) multiplied by 4(which is the amount of bytes in a float32)
let bpv=20

//2 values for the position, 0 bytes before the position values
gl.vertexAttribPointer(vertPosLocation,2,gl.FLOAT,gl.FALSE,bpv,0)

//3 values for the color, 2 values(x & y coords) * 4 bytes per value = 8 bytes before the color values
gl.vertexAttribPointer(vertColorLocation,3,gl.FLOAT,gl.FALSE,bpv,8)
 ```
The _"gl.drawArrays()"_ function call doesn't need changing, as we didn't add more vertices.<br><br>You should now see a rainbow triangle! The color of each pixel in the triangle is mixed with the nearby vertex's color value. Pixels closer to the higher corner are red, the lower left corner is green, and the lower right corner is blue. Pixels on the edge of the triangle are a blend of the colors of the 2 nearest corners, and the pixels fade to a dull gray-ish color in the middle.

## <div align='center'>Uniforms</div>

<br>In GLSL, uniforms are global values that stay the same for each draw call. Before a WebGL draw call, they can be set. WebGL has many functions for setting uniforms, as they can come in many types.<br><br>In this section, you'll learn how to create, set, and use uniforms to change your triangle.<br><br>Our goal is to make a system that allows us to scale and move our triangle around *without* changing the mesh!<br><br>Start with the code from the **Varyings & More Attributes** section. We are using uniforms to do math with the vertex position of the triangles. We are going to create 3 uniform values: a scale value, an x translation value, and a y translation value to transform the triangle. The scale value is a float, and the 2 translation values will be combined into a vec2. So inside the vertex shader...

```
let vertexShaderCode=`#version 300 es
    
    precision mediump float;
    
    in vec2 vertPos;

    in vec3 vertColor;
    
    out vec3 pixelColor;
    
    //declare a uniform with the "uniform" keyword
    
    //this is the scaling amount
    uniform float scaleAmount;
    
    //this is the translation vector
    uniform vec2 translationAmount;
    
    void main(){
        
        pixelColor=vertColor;
        
        //we multiply the "vertPos" by the "scaleAmount", and translate it by adding "translationAmount"
        gl_Position=vec4(vertPos*scaleAmount+translationAmount,0,1);
    }
`
```
That's already half of the process done! Now we need to create and set the uniform values with the WebGL.<br><br>Get the uniform by calling _gl.getUniformLocation()_, similar to getting an attribute's location.

```
//notice that the names match up to what was defined in the vertex shader

let scaleAmountLocation=gl.getUniformLocation(program,'scaleAmount')
let translationAmountLocation=gl.getUniformLocation(program,'translationAmount')
```
Now, we set the uniforms. Make sure to do this before the draw call.
```
//the values that will transform the triangle
let scaleAmount=0.3
let xTranslation=0.5
let yTranslation=0.5

//set the uniforms to the corresponding values
//use gl.uniform1f for a float, gl.uniform2f for 2 floats(a vec2)

gl.uniform1f(scaleAmountLocation,scaleAmount)
gl.uniform2f(translationAmountLocation,xTranslation,yTranslation)
```
The triangle should be smaller and moved to the up and right side! Play around with the values to change the triangles in different ways!<br><br>When setting uniforms, you can choose to pass an array into the function. GLSL supports arrays, but their usage is very limited.<br><br>Here is a list of all the types of uniforms in WebGL2, with their corresponding _"gl.uniform....."_ function:

```
gl.uniform1f (floatUniformLoc, v);                 // for float
gl.uniform1fv(floatUniformLoc, [v]);               // for float or float array
gl.uniform2f (vec2UniformLoc,  v0, v1);            // for vec2
gl.uniform2fv(vec2UniformLoc,  [v0, v1]);          // for vec2 or vec2 array
gl.uniform3f (vec3UniformLoc,  v0, v1, v2);        // for vec3
gl.uniform3fv(vec3UniformLoc,  [v0, v1, v2]);      // for vec3 or vec3 array
gl.uniform4f (vec4UniformLoc,  v0, v1, v2, v4);    // for vec4
gl.uniform4fv(vec4UniformLoc,  [v0, v1, v2, v4]);  // for vec4 or vec4 array
 
gl.uniformMatrix2fv(mat2UniformLoc, false, [  4x element array ])  // for mat2 or mat2 array
gl.uniformMatrix3fv(mat3UniformLoc, false, [  9x element array ])  // for mat3 or mat3 array
gl.uniformMatrix4fv(mat4UniformLoc, false, [ 16x element array ])  // for mat4 or mat4 array
 
gl.uniform1i (intUniformLoc,   v);                 // for int
gl.uniform1iv(intUniformLoc, [v]);                 // for int or int array
gl.uniform2i (ivec2UniformLoc, v0, v1);            // for ivec2
gl.uniform2iv(ivec2UniformLoc, [v0, v1]);          // for ivec2 or ivec2 array
gl.uniform3i (ivec3UniformLoc, v0, v1, v2);        // for ivec3
gl.uniform3iv(ivec3UniformLoc, [v0, v1, v2]);      // for ivec3 or ivec3 array
gl.uniform4i (ivec4UniformLoc, v0, v1, v2, v4);    // for ivec4
gl.uniform4iv(ivec4UniformLoc, [v0, v1, v2, v4]);  // for ivec4 or ivec4 array
 
gl.uniform1u (intUniformLoc,   v);                 // for uint
gl.uniform1uv(intUniformLoc, [v]);                 // for uint or uint array
gl.uniform2u (ivec2UniformLoc, v0, v1);            // for uvec2
gl.uniform2uv(ivec2UniformLoc, [v0, v1]);          // for uvec2 or uvec2 array
gl.uniform3u (ivec3UniformLoc, v0, v1, v2);        // for uvec3
gl.uniform3uv(ivec3UniformLoc, [v0, v1, v2]);      // for uvec3 or uvec3 array
gl.uniform4u (ivec4UniformLoc, v0, v1, v2, v4);    // for uvec4
gl.uniform4uv(ivec4UniformLoc, [v0, v1, v2, v4]);  // for uvec4 or uvec4 array
 
// for sampler2D, sampler3D, samplerCube, samplerCubeShadow, sampler2DShadow,
// sampler2DArray, sampler2DArrayShadow
gl.uniform1i (samplerUniformLoc,   v);
gl.uniform1iv(samplerUniformLoc, [v]);


@ Credits to WebGL2Fundamentals
```

## <div align='center'>Multiple Triangles</div>

<br>To draw multiple meshes, there are 2 main ways: mesh merging and multiple draw calls. These 2 methods both have their pros and cons. In this section, we'll be exploring both.<br><br>
#### <div align='center'>Mesh Merging</div>
With mesh merging, you add more vertices for triangles to a big main mesh. Starting with the code from the **Varyings & More Attributes** section, we can make an _"addTriangle()"_ function that adds more vertices to the mesh to form additional triangles.

```
let verts=[]

//params: x pos, y pos, size
function addTriangle(x,y,s){
    
    //adds 3 more vertices to the mesh via pushing to the array
    //also transforms the vertices according to the params
    verts.push(
        
        //in the middle, upwards, red color
        0+x,0.5*s+y,  1,0,0,
        
        //on the left, downwards, green color
        -0.5*s+x,-0.5*s+y,  0,1,0,
        
        //on the right, downwards, blue color
        0.5*s+x,-0.5*s+y,  0,0,1
    )
}

//adds many triangles to be rendered
addTriangle(-0.3,-0.4,0.3)
addTriangle(-0.5,0.5,0.4)
addTriangle(0.5,0.2,0.6)
```
Wait! We're not done! Only 1 triangle is showing up. Why? Remember the _"gl.drawArrays()"_ function? You have to provide the amount of vertices to be drawn!<br><br>Instead of making a counter that increments everything you call _"addTriangle"_, you can calculate the amount of vertices based on the mesh(_"verts"_ array) itself.

```
//1st param: type of primitive being drawn
//2nd param: starting vertex(often kept as 0)
//3rd param: amount of vertices

//there are 5 values per vertex. divide the total amount of values by 5 and you get the amount of vertices!
gl.drawArrays(gl.TRIANGLES,0,verts.length/5)
```
That's it! You should see 3 triangles created by the _"addTriangle"_ function.<br><br>Mesh merging is a great way to draw multiple objects, but it won't be as fast if the mesh for each object is complex and changes every frame.
<br><br>
#### <div align='center'>Multiple Draw Calls</div>
Another way to draw many objects is drawing each of them separately using 1 draw call.<br><br>You can choose to create a new mesh for each object, but that's unnecessary. Instead, you can use uniforms to change the meshes.<br><br>Start with the code from the **Uniforms** section. We can re-set the uniforms and draw again.

```
//params: x pos, y pos, size
function renderTriangle(x,y,s){

    //set the uniforms to the corresponding params
    gl.uniform1f(scaleAmountLocation,s)
    gl.uniform2f(translationAmountLocation,x,y)
    
    //1st param: type of primitive being drawn
    //2nd param: starting vertex(often kept as 0)
    //3rd param: amount of vertices
    gl.drawArrays(gl.TRIANGLES,0,3)
}

renderTriangle(0.3,-0.4,0.3)
renderTriangle(0.5,0.5,0.4)
renderTriangle(-0.5,0.2,0.6)
```
It's already done! The downside to this method is that it's unnessessary to do this for large amounts of non-changing objects every frame. It's best to use mesh merging once at the start in that scenario.<br><br>Notice that we don't need to re-bind and re-call the buffers and attribute pointers. You should always bind buffers and call _"gl.vertexAttribPointer()"_ together, and only call them if a different buffer is bound and you want to draw this mesh, otherwise it's unnessessary.

## <div align='center'>Animations</div>
<br>If you're familiar with animation frames, you can probably animate your WebGL graphics already!<br><br>You can use _"window.requestAnimationFrame()"_ to animate WebGL graphics. Start with code from the **Multiple Draw Calls** section.<br><br>You'll need to create a loop function that gets run every frame, and do delta time computations to ensure the animation is frame rate independent.

```
let dt,then=0,time=0

function loop(now){
    
    dt=(now-then)*0.001
    time+=dt
    
    //blah blah goes here
    
    then=now
    window.requestAnimationFrame(loop)
}

loop(0)
```
Because the objecst are constantly moving in an animation, drawing many triangles using multiple draw calls will be a better option!<br><br>Here, we clear the background and then redraw the triangles every frame. I use some fancy math to make the triangles move around.
```
for(let i=time;i<6.28318531+time;i+=6.28318531/12){
    
    let r=Math.sin(i*2.25)*0.75
    
    renderTriangle(Math.sin(i)*r,Math.cos(i)*r,0.4*Math.cos(i*3.5))
}
```
We also need to clear the canvas every frame. Remember to clear the canvas _before_ rendering.
```
function loop(now){
    
    dt=(now-then)*0.001*0.5
    time+=dt
    
    gl.clearColor(0.1,0,0,1)
    gl.clear(gl.COLOR_BUFFER_BIT)
    
    for(let i=time;i<6.28318531+time;i+=6.28318531/12){
        
        let r=Math.sin(i*2.25)*0.75
        
        renderTriangle(Math.sin(i)*r,Math.cos(i)*r,0.4*Math.cos(i*3.5))
    }
    
    
    then=now
    window.requestAnimationFrame(loop)
}
```
That's animation done! However, you might need to remember what parts of the code belong and don't belong in the loop to maximize performance.<br>
- It's always best to not have _"gl.getAttribLocation()"_ or _"gl.getUniformLocation()"_ calls insde the loop, but at initalization time instead.

- Buffers _can_ be set every frame, but shouldn't be created often.

- Programs should be created and shaders should be complied once at initalization. Sometimes, you may update shaders, but this is expensive.

## <div align='center'>Indexed Rendering</div>
<br>Up until this point, we've only been drawing triangles. What if we want to draw rectanles or other shapes instead?<br><br>You can draw just about anything with triangles. A rectangle can be made from 2 triangles positioned in a specific way next to each other.<br><br>Let's look at the simpliest code for a triangle, after the **Rendering a Triangle** section. We'll now create the mesh for a rectangle.
```
let verts=[
    
    //top left
    -0.6,0.5,
    //lower left
    -0.6,-0.5,
    //lower right
    0.6,-0.5,
    
    //that's 3 vertices, making 1 triangle
    //next set of vertices for the 2nd triangle:
    
    //top left
    -0.6,0.5,
    //top right
    0.6,0.5,
    //lower right
    0.6,-0.5,
]

```
Then, we update the parameter of _"gl.drawArrays()"_ to specify the new amount of vertices. There are now 6 vertices, as there are 3 per triangle, and 2 triangles.
```
//1st param: type of primitive being drawn
//2nd param: starting vertex(often kept as 0)
//3rd param: amount of vertices
gl.drawArrays(gl.TRIANGLES,0,6)
```
That'll render 2 triangles next to each other, arranged into a rectangle. You can further visualize the way the 2 triangles are oriented by offsetting their vertices by a bit:
```
let aSmallNumber=0.01

let verts=[
    
    //top left
    -0.6-aSmallNumber,0.5-aSmallNumber,
    //lower left
    -0.6-aSmallNumber,-0.5-aSmallNumber,
    //lower right
    0.6-aSmallNumber,-0.5-aSmallNumber,
    
    //that's 3 vertices, making 1 triangle
    //next set of vertices for the 2nd triangle:
    
    //top left
    -0.6+aSmallNumber,0.5+aSmallNumber,
    //top right
    0.6+aSmallNumber,0.5+aSmallNumber,
    //lower right
    0.6+aSmallNumber,-0.5+aSmallNumber,
]
```
Let's go back to the normal rectangle. Notice how the resulting rectangle has 4 vertices, but we have to use 6 to render it! There are 2 sets of 2 vertices with the same position in the mesh, but they are also mantadory in order to correctly render the triangles. The vertex shader's computations run once per vertex, so many unnessesary vertices will be bad for performance. How can we reduce the amount of excess vertices and computations while still being able to render a rectangle?<br><br>The answer is _indexed rendering_! It is a very simple and effiecent method, and should always be used whenever possible!<br><br>In indexed rendering, we provide 2 buffers. In this example, the vertex buffer stays the same, but with the excess vertices discarded. The second buffer is called the _index buffer_. It is a list of integers that specifies how vertices are connected to form triangles.<br><br>We'll start by updating our vertex array, removing the duplicate vertices. We should have 4 vertices that will become the rectangle's corners:
```
let verts=[
    
    //top left
    -0.6,0.5,
    
    //lower left
    -0.6,-0.5,
    
    //lower right
    0.6,-0.5,
    
    //top right
    0.6,0.5,
]
```
Now, we create the index buffer. Similar to the process of creating the vertex buffer, we first define an index array that uses integers to indicate how the vertices are connected.
```
//each integer refrences a specific vertex from the "verts" array
let index=[
    
    //1st triangle
    //top left, lower left, and lower right corners
    0,1,2,
    
    //2nd triangle
    //top left, lower right, and top right corners
    0,2,3
    
]
```
A very useful patern to memorize is the "0, 1, 2, 0, 2, 3" pattern. We will often see this pattern when using indexed rendering, and it'll be useful once we start making more complicated meshes.<br><br>Next, create the index buffer. The process is similar to creating the vertex buffer, bit with 2 small changes.
```
//creates the index buffer
let indexBuffer=gl.createBuffer()

//tells webgl that we are performing stuff on this specific buffer
//gl.ELEMENT_ARRAY_BUFFER indicates that this buffer is for index data
gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER,indexBuffer)

//sets data in the buffer using the mesh. the inputted data must be a Uint16Array
gl.bufferData(gl.ELEMENT_ARRAY_BUFFER,new Uint16Array(index),gl.STATIC_DRAW)
```
We're almost done! The attribute pointers and uniforms(if there are any) won't need to be changed.<br><br>Now, we update the draw call function. The _"gl.drawArrays()"_ will be replaced with the _"gl.drawElements()"_ function. Then we set the proper parameters.
```
//1st param: type of primitive being drawn
//2nd param: amount of vertices to be rendered(vertex array's length won't matter with indexed rendering)
//3rd param: the type of data(often kept as gl.UNSIGNED_SHORT)
//4th param: offset(often kept as 0)
gl.drawElements(gl.TRIANGLES,index.length,gl.UNSIGNED_SHORT,0)
```
You should see a rectangle. With indexed rendering, the amount of vertices computed is 4, with no excess duplicate vertices!











<br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>
