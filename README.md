## <div align='center'>About WebGL</div>

<br>WebGL is a rasterization API used for rendering graphics directly within a webpage, based off of OpenGL. WebGL uses JavaScript and offers many features, being widely used for 3D programs within HTML websites.<br><br>WebGL has two versions, the basic WebGL1, and WebGL2. WebGL2 is a more modern version that offers many more features compared to WebGL1. WebGL2 is backwards compatible with WebGL1 and is better, making WebGL1 pointless.<br><br>Both WebGL versions have extensions that provide additional features, but support for these extensions are quite limited. <br><br>**In this tutorial, we'll be strictly using WebGL2 and no extenstions.**<br><br>WebGL utilizes your computer's Graphics Processing Unit to rasterize pixels at lightning speeds. It takes information you provide and does math to color pixels.<br><br>Most modern browsers and machines support WebGL1. WebGL2's support may be less, but still quite high. It is supported by major browsers, but some require the experimental version.<br><br>WebGL programs run using a special shader language similar to C++, called Graphics Library Shader Language(GLSL). GLSL is a type-strict language used to write the vertex and fragment shaders of a WebGL program.<br><br>Most of the WebGL API is boilerplate code, and the rest is for rendering.

## <div align='center'>Rendering a Triangle</div>

<br>To get started with WebGL, you'll first need to prepare an HTML program. Create a canvas element that'll be used for rendering the graphics.

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
    
	//this line tells the shader what the precision is. This is needed in both the shaders. 'lowp' means low, 'mediump' means medium, and 'highp' is high.
	precision mediump float;
    
	//this is the position of the vertices, and will be passed in from the buffers later
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
Our WebGLProgram is now initialized. When you call _gl.useProgram()_, it sets the current program being used as whatever program you put in. Graphics will be rendered with that program.<br><br>Now, it's time to create the mesh and buffers. Buffers are like arrays of numbers that can only be accessed by the GPU. A "mesh" is just data that describes the geometry being drawn.<br><br>Here, we define a mesh and create a buffer:

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
Finally, we render the elements with a simple draw call! A draw call is just a call to _gl.drawArrays()_ or _gl.drawElements()_. Draw calls are quite expensive compared to the rest of the functions. You likely want less than 50 to 350 draw calls per frame for a good frame rate.

```
//1st param: type of primitive being drawn
//2nd param: starting vertex(often kept as 0)
//3rd param: amount of vertices
gl.drawArrays(gl.TRIANGLES,0,3)
```
You should now see a green triangle! It may seem like a lot, but the code won't expand much and can be reused for more complicated programs.<br><br>A draw call uses the currently bound buffer and attributes. It's good practice to bind the buffer, make the _gl.vertexAttribPointer()_ calls, and then draw.

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
In the fragment shader, we set the color based on the varying value passed in from the vertex shader.

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
gl.uniform1f (floatUniformLoc, v);   			  // for float
gl.uniform1fv(floatUniformLoc, [v]); 			  // for float or float array
gl.uniform2f (vec2UniformLoc,  v0, v1);  		  // for vec2
gl.uniform2fv(vec2UniformLoc,  [v0, v1]);    	  // for vec2 or vec2 array
gl.uniform3f (vec3UniformLoc,  v0, v1, v2);  	  // for vec3
gl.uniform3fv(vec3UniformLoc,  [v0, v1, v2]);      // for vec3 or vec3 array
gl.uniform4f (vec4UniformLoc,  v0, v1, v2, v4);	// for vec4
gl.uniform4fv(vec4UniformLoc,  [v0, v1, v2, v4]);  // for vec4 or vec4 array
 
gl.uniformMatrix2fv(mat2UniformLoc, false, [  4x element array ])  // for mat2 or mat2 array
gl.uniformMatrix3fv(mat3UniformLoc, false, [  9x element array ])  // for mat3 or mat3 array
gl.uniformMatrix4fv(mat4UniformLoc, false, [ 16x element array ])  // for mat4 or mat4 array
 
gl.uniform1i (intUniformLoc,   v);   			  // for int
gl.uniform1iv(intUniformLoc, [v]);   			  // for int or int array
gl.uniform2i (ivec2UniformLoc, v0, v1);  		  // for ivec2
gl.uniform2iv(ivec2UniformLoc, [v0, v1]);    	  // for ivec2 or ivec2 array
gl.uniform3i (ivec3UniformLoc, v0, v1, v2);  	  // for ivec3
gl.uniform3iv(ivec3UniformLoc, [v0, v1, v2]);      // for ivec3 or ivec3 array
gl.uniform4i (ivec4UniformLoc, v0, v1, v2, v4);	// for ivec4
gl.uniform4iv(ivec4UniformLoc, [v0, v1, v2, v4]);  // for ivec4 or ivec4 array
 
gl.uniform1u (intUniformLoc,   v);   			  // for uint
gl.uniform1uv(intUniformLoc, [v]);   			  // for uint or uint array
gl.uniform2u (ivec2UniformLoc, v0, v1);  		  // for uvec2
gl.uniform2uv(ivec2UniformLoc, [v0, v1]);    	  // for uvec2 or uvec2 array
gl.uniform3u (ivec3UniformLoc, v0, v1, v2);  	  // for uvec3
gl.uniform3uv(ivec3UniformLoc, [v0, v1, v2]);      // for uvec3 or uvec3 array
gl.uniform4u (ivec4UniformLoc, v0, v1, v2, v4);	// for uvec4
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
Another way to draw many objects is drawing each of them separately using 1 draw call.<br><br>You can choose to create a new mesh for each object, but that's unnecessary. Instead, you can use uniforms to change the meshes.<br><br>Start with the code from the **Uniforms** section. We can reset the uniforms and draw again.

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
It's already done! The downside to this method is that it's unnecessary to do this for large amounts of non-changing objects every frame. It's best to use mesh merging once at the start in that scenario.<br><br>Notice that we don't need to re-bind and re-call the buffers and attribute pointers. You should always bind buffers and call _"gl.vertexAttribPointer()"_ together, and only call them if a different buffer is bound and you want to draw this mesh, otherwise it's unnecessary.

## <div align='center'>Animations</div>
<br>If you're familiar with animation frames, you can probably animate your WebGL graphics already!<br><br>You can use _"window.requestAnimationFrame()"_ to animate WebGL graphics. Start with code from the **Multiple Draw Calls** section.<br><br>You'll need to create a loop function that gets run every frame, and do delta time computations to ensure the animation is frame rate independent.

```
let dt,then=0,time=0

function loop(now){
    
	dt=Math.min((now-then)*0.001,0.1)
	time+=dt
    
	//blah blah goes here
    
	then=now
	window.requestAnimationFrame(loop)
}

loop(0)
```
Because the objects are constantly moving in an animation, drawing many triangles using multiple draw calls will be a better option!<br><br>Here, we clear the background and then redraw the triangles every frame. I use some fancy math to make the triangles move around.
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
- It's always best to not have _"gl.getAttribLocation()"_ or _"gl.getUniformLocation()"_ calls inside the loop, but at initialization time instead.

- Buffers _can_ be set every frame, but shouldn't be created often.

- Programs should be created and shaders should be compiled once at initialization. Sometimes, you may update shaders, but this is expensive.

## <div align='center'>Indexed Rendering</div>
<br>Up until this point, we've only been drawing triangles. What if we want to draw rectangles or other shapes instead?<br><br>You can draw just about anything with triangles. A rectangle can be made from 2 triangles positioned in a specific way next to each other.<br><br>Let's look at the simplest code for a triangle, after the **Rendering a Triangle** section. We'll now create the mesh for a rectangle.
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
Let's go back to the normal rectangle. Notice how the resulting rectangle has 4 vertices, but we have to use 6 to render it! There are 2 sets of 2 vertices with the same position in the mesh, but they are also mandatory in order to correctly render the triangles. The vertex shader's computations run once per vertex, so many unnecessary vertices will be bad for performance. How can we reduce the amount of excess vertices and computations while still being able to render a rectangle?<br><br>The answer is _indexed rendering_! It is a very simple and efficient method, and should always be used whenever possible!<br><br>In indexed rendering, we provide 2 buffers. In this example, the vertex buffer stays the same, but with the excess vertices discarded. The second buffer is called the _index buffer_. It is a list of integers that specifies how vertices are connected to form triangles.<br><br>We'll start by updating our vertex array, removing the duplicate vertices. We should have 4 vertices that will become the rectangle's corners:
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
//each integer references a specific vertex from the "verts" array
let index=[
    
	//1st triangle
	//top left, lower left, and lower right corners
	0,1,2,
    
	//2nd triangle
	//top left, lower right, and top right corners
	0,2,3
    
]
```
A very useful pattern to memorize is the "0, 1, 2, 0, 2, 3" pattern. We will often see this pattern when using indexed rendering, and it'll be useful once we start making more complicated meshes.<br><br>Next, create the index buffer. The process is similar to creating the vertex buffer, but with 2 small changes.
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

## <div align='center'>2D Matrices & Transformations</div>
<br>In this section, we're going to learn how matrices work and how to apply transformations with them.<br><br>Matrices are basically grids of numbers that can be used to transform vectors(translate, scale, rotate). In 2D programs, 3x3 matrices are used as they are able to perform translation transformations. 2x2 matrices can only scale and rotate in 2D.<br><br>Get started with the code from the **Indexed Rendering** section. We are first going to add vertex colors(see **Varyings & More Attributes**) and a matrix uniform.<br><br>Updated shaders:

```
let vertexShaderCode=`#version 300 es
    
	precision mediump float;
    
	in vec2 vertPos;
	in vec3 vertColor;
    
	//the matrix for transformations
	uniform mat3 modelMatrix;
    
	out vec3 pixColor;
    
	void main(){
      
  	  pixColor=vertColor;
      
  	  //transform the vec2 by turning it into a vec3 then multiply with the mat3
  	  //gl_Position needs to be a vec4 so we add another useless component
  	  gl_Position=vec4(modelMatrix*vec3(vertPos,1),1);
	}
`

let fragmentShaderCode=`#version 300 es
    
	precision mediump float;
    
	in vec3 pixColor;
    
	out vec4 fragColor;
    
	void main(){
      
  	  fragColor=vec4(pixColor,1);
	}
`
```
Updated mesh:
```
let verts=[
    
	//top left, red
	-0.5,0.5,   1,0,0,
	//bottom left, green
	-0.5,-0.5,  0,1,0,
	//bottom right, blue
	0.5,-0.5,  0,0,1,
	//top right front, yellow
	0.5,0.5,  1,1,0,
]

let index=[
    
	//front side
	0,1,2,
	0,2,3
    
]
```
Buffer creation stays the same. Updated attribute locations and pointers:
```
let vertPosLocation=gl.getAttribLocation(program,'vertPos')
gl.enableVertexAttribArray(vertPosLocation)

let vertColorLocation=gl.getAttribLocation(program,'vertColor')
gl.enableVertexAttribArray(vertColorLocation)

//bytes per vertex. the total amount of values per a vertex(now it's 5(x,y,r,g,b)) multiplied by 4(which is the amount of bytes in a float32)
let bpv=20

//2 values for the position, 0 bytes before the position values
gl.vertexAttribPointer(vertPosLocation,2,gl.FLOAT,gl.FALSE,bpv,0)

//3 values for the color, 2 values(x & y coords) * 4 bytes per value = 8 bytes before the color values
gl.vertexAttribPointer(vertColorLocation,3,gl.FLOAT,gl.FALSE,bpv,8)
```
Getting and setting the matrix uniform:
```
let modelMatrixLocation=gl.getUniformLocation(program,'modelMatrix')

//currently an identity matrix, which applies no transformations
let modelMatrix=new Float32Array([
    
	1,0,0,
	0,1,0,
	0,0,1
])

//use gl.uniformMatrix3fv for 3x3 matrices
gl.uniformMatrix3fv(modelMatrixLocation,false,modelMatrix)
```
After adding the changes, you should see a multi-colored square.<br><br>Now, we're going to get started with matrix math. You can perform simple operations like addition, subtraction, and multiplication on matrices.<br><br>In WebGL graphics, only matrix multiplication is required. Multiplying matrices "stack" transformations on top of each other. The code for multiplying 2 3x3 matrices follows:
```
//params "out": the output matrix(array)
//params "a": a matrix to be multiplied with "b"
//params "b": a matrix to be multiplied with "a"

function mult3x3Mat(out, a, b) {
    
	let a00 = a[0],
  	  a01 = a[1],
  	  a02 = a[2];
    
	let a10 = a[3],
  	  a11 = a[4],
  	  a12 = a[5];
    
	let a20 = a[6],
  	  a21 = a[7],
  	  a22 = a[8];
    
	let b00 = b[0],
  	  b01 = b[1],
  	  b02 = b[2];
    
	let b10 = b[3],
  	  b11 = b[4],
  	  b12 = b[5];
    
	let b20 = b[6],
  	  b21 = b[7],
  	  b22 = b[8];
    
	out[0] = b00 * a00 + b01 * a10 + b02 * a20;
	out[1] = b00 * a01 + b01 * a11 + b02 * a21;
	out[2] = b00 * a02 + b01 * a12 + b02 * a22;
	out[3] = b10 * a00 + b11 * a10 + b12 * a20;
	out[4] = b10 * a01 + b11 * a11 + b12 * a21;
	out[5] = b10 * a02 + b11 * a12 + b12 * a22;
	out[6] = b20 * a00 + b21 * a10 + b22 * a20;
	out[7] = b20 * a01 + b21 * a11 + b22 * a21;
	out[8] = b20 * a02 + b21 * a12 + b22 * a22;
    
	return out;
}


@ Credits to glMatrix
```
To apply transformations, you can directly perform a transformation on a matrix, or multiply a matrix with another matrix with the the transformations already in place.<br><br>Often, it's best to reduce matrix operations for performance, by computing them once with JS, and then uniforming them to the GPU. The technique of directly applying transformations on a single matrix is favored. It is done by manually optimizing and simplifying the process of creating an alternate matrix with the transformations and multiplying. This means that matrix multiplication may be used less often or not at all.<br><br>Small note: matrix naming may vary a lot. There are many ways to name matrices depending on what information they contain. It's often best to just stick to your own naming preferences.<br><br>Here are ways to transform 3x3 matrices:
```
function translate3x3Mat(out, a, x, y) {
    
	let a00 = a[0],
  	  a01 = a[1],
  	  a02 = a[2],
  	  a10 = a[3],
  	  a11 = a[4],
  	  a12 = a[5],
  	  a20 = a[6],
  	  a21 = a[7],
  	  a22 = a[8];
    
	out[0] = a00;
	out[1] = a01;
	out[2] = a02;
	out[3] = a10;
	out[4] = a11;
	out[5] = a12;
	out[6] = x * a00 + y * a10 + a20;
	out[7] = x * a01 + y * a11 + a21;
	out[8] = x * a02 + y * a12 + a22;
    
	return out;
}

function rotate3x3Mat(out, a, rad) {
    
	let a00 = a[0],
  	  a01 = a[1],
  	  a02 = a[2],
  	  a10 = a[3],
  	  a11 = a[4],
  	  a12 = a[5],
  	  a20 = a[6],
  	  a21 = a[7],
  	  a22 = a[8],
  	  s = Math.sin(rad),
  	  c = Math.cos(rad);
    
	out[0] = c * a00 + s * a10;
	out[1] = c * a01 + s * a11;
	out[2] = c * a02 + s * a12;
	out[3] = c * a10 - s * a00;
	out[4] = c * a11 - s * a01;
	out[5] = c * a12 - s * a02;
	out[6] = a20;
	out[7] = a21;
	out[8] = a22;
    
	return out;
}

function scale3x3Mat(out, a, x, y) {

	out[0] = x * a[0];
	out[1] = x * a[1];
	out[2] = x * a[2];
	out[3] = y * a[3];
	out[4] = y * a[4];
	out[5] = y * a[5];
	out[6] = a[6];
	out[7] = a[7];
	out[8] = a[8];
    
	return out;
}


@ Credits to glMatrix
```
Because matrix transformations stack on top of each other, transformations applied are affected by previous transformations. For example, if you translate a mesh and rotate it, it will rotate about its center. If you rotate it then translate it, it will be rotated then translated, but the translation vector will be rotated too. This is the same for transformations done by matrix multiplication(they are the same process, but multiplication is simplified into the direct transformations).<br><br>With the functions in place, you can now transform your geometry. Here's an example of what you can do to the model matrix:
```
let x=0.3,
	y=-0.3,
	rot=3.5,
	sx=0.7,
	sy=0.5

translate3x3Mat(modelMatrix,modelMatrix,x,y)
rotate3x3Mat(modelMatrix,modelMatrix,rot)
scale3x3Mat(modelMatrix,modelMatrix,sx,sy)
```
You can change the numbers to see the effects on the mesh. You can also change the mesh by reordering the transformation operations.<br><br>Important!
- The "default" matrix is called an identity matrix, and goes like
```
[
	1,0,0,
	0,1,0,
	0,0,1
]
```
as you already saw above. It applies no transformations with transforming a vector and does nothing when multiplied with a matrix. Remember this!
- When providing a matrix as a uniform, make sure the supplied matrix is a typed array! Float32Array is very commonly used. This will result in a large performance boost, possible up to x9 speed!

## <div align='center'>3D Transformations</div>

<br>This section is the first part of 3D graphics. The next part is the **3D Graphics** section.<br><br>3D graphics are quite simple if you can understand the matrix math. Essentially, matrix transformation moves, rotates, and projects vertices, resulting in a new position vector. Perspective is applied automatically by WebGL, and that's basically it.<br><br>The complicated parts of 3D graphics with WebGL(especially WebGL2) are the techniques you can utilize to improve quality and performance.<br><br>To get started with 3D transformations, we need to make a 3D mesh. Starting with the code from the **2D Matrices & Transformations** section, turn the attribute _"vertPos"_ into a vec3 and supply another number(the z position) for each vertex's position. We also need to turn the mat3 _"modelMatrix"_ into a mat4.
```
let vertexShaderCode=`#version 300 es
    
	precision mediump float;
    
	//vertPos is now a vec3! x, y, and z values!
	in vec3 vertPos;
    
	in vec3 vertColor;
    
	//the model matrix is now a mat4! 4x4 matrices are needed for 3D stuff
	uniform mat4 modelMatrix;
    
	out vec3 pixColor;
    
	void main(){
      
  	  pixColor=vertColor;
      
  	  //transform the vec3 by turning it into a vec4 then multiply with the mat4
  	  gl_Position=modelMatrix*vec4(vertPos,1);
	}
`
```
The new vertex buffer, now with z positions for each vertex. The z positions right now are set as -0.5, which is a bit closer to the camera.
```
let verts=[
    
	//top left front, red
	-0.5,0.5,-0.5,   1,0,0,
	//bottom left front, green
	-0.5,-0.5,-0.5,  0,1,0,
	//bottom right front, blue
	0.5,-0.5,-0.5,  0,0,1,
	//top right front, yellow
	0.5,0.5,-0.5,  1,1,0,
]
```
Buffer creation stays the same, of course. Now we update the attribute pointers to specify the new format of the mesh's vertices.
```
//bytes per vertex. the total amount of values per a vertex(now it's 6(x,y,z,r,g,b)) multiplied by 4(which is the amount of bytes in a float32)
let bpv=24

//3 values for the position, 0 bytes before the position values
gl.vertexAttribPointer(vertPosLocation,3,gl.FLOAT,gl.FALSE,bpv,0)

//3 values for the color, 3 values(x & y & z coords) * 4 bytes per value = 12 bytes before the color values
gl.vertexAttribPointer(vertColorLocation,3,gl.FLOAT,gl.FALSE,bpv,12)
```
The _"modelMatrix"_ is now a 4x4 matrix. We set it to an identity matrix for now.
```
//currently an identity matrix, which applies no transformations
let modelMatrix=new Float32Array([
    
	1,0,0,0,
	0,1,0,0,
	0,0,1,0,
	0,0,0,1
])

//use gl.uniformMatrix4fv for 4x4 matrices
gl.uniformMatrix4fv(modelMatrixLocation,false,modelMatrix)
```
The _"gl.drawElements()"_ function stays the same. Now, you should see the same square again.<br><br>Transformations with 4x4 matrices are similar to 3x3 ones. We use 3x3 matrices for 2D, and then 4x4 matrices for 3D. In 3D, transformations and matrix operations are nearly the same.<br><br>Scale and translation transformations now use x, y, and z values or all the axes, instead of just x and y. However, in 3D, you have 3 axes you can rotate objects about. Here is the code for each types of useful 4x4 operations and transformations:
```
function mult4x4Mat(out, a, b) {

	let a00 = a[0],
  	  a01 = a[1],
  	  a02 = a[2],
  	  a03 = a[3];
    
	let a10 = a[4],
  	  a11 = a[5],
  	  a12 = a[6],
  	  a13 = a[7];
    
	let a20 = a[8],
  	  a21 = a[9],
  	  a22 = a[10],
  	  a23 = a[11];
    
	let a30 = a[12],
  	  a31 = a[13],
  	  a32 = a[14],
  	  a33 = a[15];
    
	let b0 = b[0],
  	  b1 = b[1],
  	  b2 = b[2],
  	  b3 = b[3];
    
	out[0] = b0 * a00 + b1 * a10 + b2 * a20 + b3 * a30;
	out[1] = b0 * a01 + b1 * a11 + b2 * a21 + b3 * a31;
	out[2] = b0 * a02 + b1 * a12 + b2 * a22 + b3 * a32;
	out[3] = b0 * a03 + b1 * a13 + b2 * a23 + b3 * a33;
    
	b0 = b[4];
	b1 = b[5];
	b2 = b[6];
	b3 = b[7];
    
	out[4] = b0 * a00 + b1 * a10 + b2 * a20 + b3 * a30;
	out[5] = b0 * a01 + b1 * a11 + b2 * a21 + b3 * a31;
	out[6] = b0 * a02 + b1 * a12 + b2 * a22 + b3 * a32;
	out[7] = b0 * a03 + b1 * a13 + b2 * a23 + b3 * a33;
    
	b0 = b[8];
	b1 = b[9];
	b2 = b[10];
	b3 = b[11];
    
	out[8] = b0 * a00 + b1 * a10 + b2 * a20 + b3 * a30;
	out[9] = b0 * a01 + b1 * a11 + b2 * a21 + b3 * a31;
	out[10] = b0 * a02 + b1 * a12 + b2 * a22 + b3 * a32;
	out[11] = b0 * a03 + b1 * a13 + b2 * a23 + b3 * a33;
    
	b0 = b[12];
	b1 = b[13];
	b2 = b[14];
	b3 = b[15];
    
	out[12] = b0 * a00 + b1 * a10 + b2 * a20 + b3 * a30;
	out[13] = b0 * a01 + b1 * a11 + b2 * a21 + b3 * a31;
	out[14] = b0 * a02 + b1 * a12 + b2 * a22 + b3 * a32;
	out[15] = b0 * a03 + b1 * a13 + b2 * a23 + b3 * a33;
    
	return out;
}

function translate4x4Mat(out, a, x, y, z) {
    
	let a00, a01, a02, a03;
	let a10, a11, a12, a13;
	let a20, a21, a22, a23;
    
	if (a === out) {
      
  	  out[12] = a[0] * x + a[4] * y + a[8] * z + a[12];
  	  out[13] = a[1] * x + a[5] * y + a[9] * z + a[13];
  	  out[14] = a[2] * x + a[6] * y + a[10] * z + a[14];
  	  out[15] = a[3] * x + a[7] * y + a[11] * z + a[15];
      
	} else {
      
  	  a00 = a[0];
  	  a01 = a[1];
  	  a02 = a[2];
  	  a03 = a[3];
  	  a10 = a[4];
  	  a11 = a[5];
  	  a12 = a[6];
  	  a13 = a[7];
  	  a20 = a[8];
  	  a21 = a[9];
  	  a22 = a[10];
  	  a23 = a[11];
  	  out[0] = a00;
  	  out[1] = a01;
  	  out[2] = a02;
  	  out[3] = a03;
  	  out[4] = a10;
  	  out[5] = a11;
  	  out[6] = a12;
  	  out[7] = a13;
  	  out[8] = a20;
  	  out[9] = a21;
  	  out[10] = a22;
  	  out[11] = a23;
  	  out[12] = a00 * x + a10 * y + a20 * z + a[12];
  	  out[13] = a01 * x + a11 * y + a21 * z + a[13];
  	  out[14] = a02 * x + a12 * y + a22 * z + a[14];
  	  out[15] = a03 * x + a13 * y + a23 * z + a[15];
      
	}
    
	return out;
}

function rotateX4x4Mat(out, a, rad) {
    
	let s = Math.sin(rad);
	let c = Math.cos(rad);
	let a10 = a[4];
	let a11 = a[5];
	let a12 = a[6];
	let a13 = a[7];
	let a20 = a[8];
	let a21 = a[9];
	let a22 = a[10];
	let a23 = a[11];
    
	if (a !== out) {
      
  	  out[0] = a[0];
  	  out[1] = a[1];
  	  out[2] = a[2];
  	  out[3] = a[3];
  	  out[12] = a[12];
  	  out[13] = a[13];
  	  out[14] = a[14];
  	  out[15] = a[15];
	}
    
	out[4] = a10 * c + a20 * s;
	out[5] = a11 * c + a21 * s;
	out[6] = a12 * c + a22 * s;
	out[7] = a13 * c + a23 * s;
	out[8] = a20 * c - a10 * s;
	out[9] = a21 * c - a11 * s;
	out[10] = a22 * c - a12 * s;
	out[11] = a23 * c - a13 * s;
    
	return out;
}

function rotateY4x4Mat(out, a, rad) {
    
	let s = Math.sin(rad);
	let c = Math.cos(rad);
    
	let a00 = a[0];
	let a01 = a[1];
	let a02 = a[2];
	let a03 = a[3];
	let a20 = a[8];
	let a21 = a[9];
	let a22 = a[10];
	let a23 = a[11];
    
	if (a !== out) {
      
  	  out[4] = a[4];
  	  out[5] = a[5];
  	  out[6] = a[6];
  	  out[7] = a[7];
  	  out[12] = a[12];
  	  out[13] = a[13];
  	  out[14] = a[14];
  	  out[15] = a[15];
	}
    
	out[0] = a00 * c - a20 * s;
	out[1] = a01 * c - a21 * s;
	out[2] = a02 * c - a22 * s;
	out[3] = a03 * c - a23 * s;
	out[8] = a00 * s + a20 * c;
	out[9] = a01 * s + a21 * c;
	out[10] = a02 * s + a22 * c;
	out[11] = a03 * s + a23 * c;
    
	return out;
}

function rotateZ4x4Mat(out, a, rad) {

	let s = Math.sin(rad);
	let c = Math.cos(rad);
    
	let a00 = a[0];
	let a01 = a[1];
	let a02 = a[2];
	let a03 = a[3];
	let a10 = a[4];
	let a11 = a[5];
	let a12 = a[6];
	let a13 = a[7];
    
	if (a !== out) {
      
  	  out[8] = a[8];
  	  out[9] = a[9];
  	  out[10] = a[10];
  	  out[11] = a[11];
  	  out[12] = a[12];
  	  out[13] = a[13];
  	  out[14] = a[14];
  	  out[15] = a[15];
	}
    
	out[0] = a00 * c + a10 * s;
	out[1] = a01 * c + a11 * s;
	out[2] = a02 * c + a12 * s;
	out[3] = a03 * c + a13 * s;
	out[4] = a10 * c - a00 * s;
	out[5] = a11 * c - a01 * s;
	out[6] = a12 * c - a02 * s;
	out[7] = a13 * c - a03 * s;
    
	return out;
}

function scale4x4Mat(out, a, x, y, z) {

	out[0] = a[0] * x;
	out[1] = a[1] * x;
	out[2] = a[2] * x;
	out[3] = a[3] * x;
	out[4] = a[4] * y;
	out[5] = a[5] * y;
	out[6] = a[6] * y;
	out[7] = a[7] * y;
	out[8] = a[8] * z;
	out[9] = a[9] * z;
	out[10] = a[10] * z;
	out[11] = a[11] * z;
	out[12] = a[12];
	out[13] = a[13];
	out[14] = a[14];
	out[15] = a[15];
    
	return out;
}


@ Credits to glMatrix
```
With the transformation functions in place, you can now transform the mesh.
```
let rx=0,
	ry=0.8,
	rz=0

rotateX4x4Mat(modelMatrix,modelMatrix,rx)
rotateY4x4Mat(modelMatrix,modelMatrix,ry)
rotateZ4x4Mat(modelMatrix,modelMatrix,rz)
```
It's hard to tell what the 3D transformations do, so let's animate it.
```
let dt,then=0,time=0

function loop(now){
    
	dt=(now-then)*0.001
	time+=dt
    
	gl.clearColor(0.1,0,0,1)
	gl.clear(gl.COLOR_BUFFER_BIT)
    
	let modelMatrix=new Float32Array([
      
  	  1,0,0,0,
  	  0,1,0,0,
  	  0,0,1,0,
  	  0,0,0,1
	])
    
	let rx=time*0.1,
  	  ry=time,
  	  rz=0
    
	rotateX4x4Mat(modelMatrix,modelMatrix,rx)
	rotateY4x4Mat(modelMatrix,modelMatrix,ry)
	rotateZ4x4Mat(modelMatrix,modelMatrix,rz)
    
	gl.uniformMatrix4fv(modelMatrixLocation,false,modelMatrix)
    
	gl.drawElements(gl.TRIANGLES,index.length,gl.UNSIGNED_SHORT,0)
    
    
	then=now
	window.requestAnimationFrame(loop)
}

loop(0)
```
It's still quite difficult to see, but with some imagination, you can make a sense of what is happening.<br><br>In the next section, we'll add cameras and perspective to create actual 3D graphics.

## <div align='center'>3D Graphics</div>

<br>In this section, we are going to create a basic cube.<br><br>First of all, a vital part of 3D programs are the backface culling and depth testing features. Backface culling gets rid of unnecessary triangles facing away from the camera, which will speed up rendering by 2x! Depth testing decides if a fragment should be drawn if there is an existing fragment in front of it. This will get rid of weird clipping effects.<br><br>You can enable these features anywhere outside the render loop:
```
gl.enable(gl.CULL_FACE)
gl.enable(gl.DEPTH_TEST)


//this part below is optional! these features are the default settings
//it's unlikely you'll need to use settings other than these

gl.cullFace(gl.BACK)
gl.depthFunc(gl.LEQUAL)
```
You will also need to update the _"gl.clear()"_ function:
```
gl.clear(gl.COLOR_BUFFER_BIT|gl.DEPTH_BUFFER_BIT)
```
It's time to add a camera to properly render the scene.<br><br>There are 2 main types of 3D projection: perspective and orthogonal. Orthogonal projection makes objects farther away appear at the same size, unlike perspective projection. Perspective projection resembles our eyes in real life, where distant objects shrink in size.<br><br>Projection is done in the vertex shader by multiplying the vertex transformation matrix with the projection matrix. The transformed vector is a vec4 with the w component containing the depth of the object. The WebGL pipeline automatically divides the set _"gl\_Position"_ value with _"gl\_Position.w"_. This is called perspective division.<br><br>Since most 3D programs use realistic perspective projection, we'll use that too. However, changing between the types of projection is easy, as you only need to change the projection matrix. Here's the matrix for perspective projection:
```
//fov: field of view in degrees
//aspect: canvas width divided by height
//zn: nearest distance camera can render
//zf: farthest distance camera can render
//make sure zn is not 0 and zf is not a giant number(1,000 is often fine)!
//limiting the distance you render and help with performance

function perspectiveMat(fov,aspect,zn,zf){
    
	let f=Math.tan(1.57079632679-fov*0.008726646),
  	  rangeInv=1/(zn-zf)
      
	return new Float32Array([
  	  f/aspect,0,0,0,
  	  0,f,0,0,
  	  0,0,(zn+zf)*rangeInv,-1,
  	  0,0,zn*zf*rangeInv*2,0
	])
}
```
Create the projection matrix:
```
let projectionMatrix=perspectiveMat(60,canvas.width/canvas.height,0.1,1000)
```
The updated _"modelMatrix"_ in the render loop:
```
//notice that here we apply translation before rotation!
//normally, we rotate first, and then translate to create an actual camera transformation matrix(given rotation and position)
//here, translation is applied first so that the camera "orbits" around the shape given the rotation
translate4x4Mat(modelMatrix,modelMatrix,0,0,-3)

rotateX4x4Mat(modelMatrix,modelMatrix,rx)
rotateY4x4Mat(modelMatrix,modelMatrix,ry)
rotateZ4x4Mat(modelMatrix,modelMatrix,rz)

//matrix multiplication isn't commutative! mat1*mat2 != mat2*mat1
//switch the "projectionMatrix" and "modelMatrix" multiplication around and stuff will break!
mult4x4Mat(modelMatrix,projectionMatrix,modelMatrix)
```
Almost done! Now, the only thing left to do is to create a cube mesh to render:
```
let verts=[
    
	//front side
	-0.5,0.5,-0.5,  0,1,0,
	-0.5,-0.5,-0.5,  0,1,0,
	0.5,-0.5,-0.5,  0,1,0,
	0.5,0.5,-0.5,  0,1,0,
    
	//back side
	-0.5,0.5,0.5,  1,1,0,
	-0.5,-0.5,0.5,  1,1,0,
	0.5,-0.5,0.5,  1,1,0,
	0.5,0.5,0.5,  1,1,0,
    
	//top side
	-0.5,0.5,0.5,  1,0,0,
	-0.5,0.5,-0.5,  1,0,0,
	0.5,0.5,-0.5,  1,0,0,
	0.5,0.5,0.5,  1,0,0,
    
	//bottom side
	-0.5,-0.5,0.5,  1,0,1,
	-0.5,-0.5,-0.5,  1,0,1,
	0.5,-0.5,-0.5,  1,0,1,
	0.5,-0.5,0.5,  1,0,1,
    
	//left side
	-0.5,0.5,-0.5,  0,0,1,
	-0.5,-0.5,-0.5,  0,0,1,
	-0.5,-0.5,0.5,  0,0,1,
	-0.5,0.5,0.5,  0,0,1,
    
	//right side
	0.5,0.5,-0.5,  0,1,1,
	0.5,-0.5,-0.5,  0,1,1,
	0.5,-0.5,0.5,  0,1,1,
	0.5,0.5,0.5,  0,1,1,
]

let index=[
    
	//front side
	2,1,0,
	3,2,0,
    
	//back side
	4,5,6,
	4,6,7,
    
	//top side
	10,9,8,
	11,10,8,
    
	//bottom side
	12,13,14,
	12,14,15,
    
	//left side
	16,17,18,
	16,18,19,
    
	//right side
	22,21,20,
	23,22,20,
]
```
It's done! You should see a spinning cube with differently colored sides! Notice that in the mesh, we have to make separate vertices for each side of the cube, each with their own color. However, with indexed rendering, we still are saving computations as we only need 4 vertices for each side(as opposed to 6). 4 vertices are shared within the 2 triangles for each face and so 2 are reused.<br><br>In the next section, we will learn how to apply lighting.

## <div align='center'>Lighting and Normals</div>

<br>In 3D graphics, normals are essential in lighting computations. A normal is a normalized vector that points in the direction that a surface is facing in. At any given pixel of a rendered mesh, the face the pixel is in has a direction it's pointing towards.<br><br>There's no reliable and simple way to compute what a normal vector is at a point, so we need to provide normals from the mesh. We can transfer the provided normal vector into the fragment shader using varyings. This will also be beneficial for more complicated meshes where surfaces need to look smooth.<br><br>Let's start with the code from the **3D Graphics** section. First, we will make the mesh faces the same color to see the shading better. Then, add the normal of the face that each vertex is in. The format of the vertex array is now x1,y1,z1,r1,g1,b1,nx1,ny1,nz1,x2,y2,z2,r2... and so on
```
let verts=[
    
	//front side, normal faces towards
	-0.5,0.5,-0.5,  0,1,0,  0,0,-1,
	-0.5,-0.5,-0.5,  0,1,0,  0,0,-1,
	0.5,-0.5,-0.5,  0,1,0,  0,0,-1,
	0.5,0.5,-0.5,  0,1,0,  0,0,-1,
    
	//back side, normal faces away
	-0.5,0.5,0.5,  0,1,0,  0,0,1,
	-0.5,-0.5,0.5,  0,1,0,  0,0,1,
	0.5,-0.5,0.5,  0,1,0,  0,0,1,
	0.5,0.5,0.5,  0,1,0,  0,0,1,
    
	//top side, normal faces up
	-0.5,0.5,0.5,  0,1,0,  0,1,0,
	-0.5,0.5,-0.5,  0,1,0,  0,1,0,
	0.5,0.5,-0.5,  0,1,0,  0,1,0,
	0.5,0.5,0.5,  0,1,0,  0,1,0,
    
	//bottom side, normal faces down
	-0.5,-0.5,0.5,  0,1,0,  0,-1,0,
	-0.5,-0.5,-0.5,  0,1,0,  0,-1,0,
	0.5,-0.5,-0.5,  0,1,0,  0,-1,0,
	0.5,-0.5,0.5,  0,1,0,  0,-1,0,
    
	//left side, normal faces left
	-0.5,0.5,-0.5,  0,1,0,  -1,0,0,
	-0.5,-0.5,-0.5,  0,1,0,  -1,0,0,
	-0.5,-0.5,0.5,  0,1,0,  -1,0,0,
	-0.5,0.5,0.5,  0,1,0,  -1,0,0,
    
	//right side, normal faces right
	0.5,0.5,-0.5,  0,1,0,  1,0,0,
	0.5,-0.5,-0.5,  0,1,0,  1,0,0,
	0.5,-0.5,0.5,  0,1,0,  1,0,0,
	0.5,0.5,0.5,  0,1,0,  1,0,0,
]
```
Get the normal attribute's location(we will add it to the vertex shader later):
```
let vertNormalLocation=gl.getAttribLocation(program,'vertNormal')
gl.enableVertexAttribArray(vertNormalLocation)
```
Updated vertex attribute pointers, adjusted for the new normal attributes:
```
//bytes per vertex. the total amount of values per a vertex(now it's 9(x,y,z,r,g,b,nx,ny,nz)) multiplied by 4(which is the amount of bytes in a float32)
let bpv=36

//3 values for the position, 0 bytes before the position values
gl.vertexAttribPointer(vertPosLocation,3,gl.FLOAT,gl.FALSE,bpv,0)

//3 values for the color, 3 values(x & y & z coords) * 4 bytes per value = 12 bytes before the color values
gl.vertexAttribPointer(vertColorLocation,3,gl.FLOAT,gl.FALSE,bpv,12)

//3 values for the normal, 6 values(x & y & z & nx & ny & nz coords) * 4 bytes per value = 24 bytes before the color values
gl.vertexAttribPointer(vertNormalLocation,3,gl.FLOAT,gl.FALSE,bpv,24)
```
Here's the new vertex shader, with the normal attribute and varying added:
```
let vertexShaderCode=`#version 300 es
    
	precision mediump float;
    
	in vec3 vertPos;
	in vec3 vertColor;
    
	//the vertex normal's attribute
	in vec3 vertNormal;
    
	uniform mat4 modelMatrix;
    
	out vec3 pixColor;
    
	//transfer the normal to the fragment shader using a varying
	out vec3 pixNormal;
    
	void main(){
      
  	  pixColor=vertColor;
  	  pixNormal=vertNormal;
      
  	  gl_Position=modelMatrix*vec4(vertPos,1);
	}
`
```
To compute lighting, we need to understand how dot products work. A dot product of 2 vec3s is computed as _"a.x\*b.x+a.y\*b.y+a.z\*b.z"_. If the 2 vectors are normalized, the dot product will be the cosine of the angle between them. Basically, if we normalize 2 vectors and take their dot product, the result is a number ranging from [-1,1] that tells how close they are to facing each other.<br><br>If 2 normalized vectors are in the same direction, the dot product is 1. If they are completely opposite of each other, the dot product is -1. If they are perpendicular, the dot product is 0.<br><br>We can use this number to multiply it by the fragment's color to darken it based on the normal and light direction. This is called directional lighting, because the light source has no point, but illuminates objects from a single direction.<br><br>Here's the fragment shader that applies directional lighting:
```
let fragmentShaderCode=`#version 300 es
    
	precision mediump float;
    
	in vec3 pixColor;
    
	//the vertex normal varying, transferred from the vertex shader
	in vec3 pixNormal;
    
	out vec4 fragColor;
    
	void main(){
      
  	  //the direction of the light
  	  vec3 lightDir=normalize(vec3(-0.7,-1.5,-1));
      
  	  //a measure of how different normal and light dir are(in terms of direction)
  	  //the dot product ranges from [-1,1] so the "*0.5+0.5" remaps it to the [0,1] range
  	  //faces facing directly away from the light will be lit 0%, and if towards the light, up to 100%
  	  float diffuse=dot(-lightDir,pixNormal)*0.5+0.5;
      
  	  //because "diffuse" is between [0,1], multiplying it by the color values will decrease them, therefore darkening the color
  	  fragColor=vec4(pixColor*diffuse,1);
	}
`    
```
That's it for basic directional lighting! Your green box's sides should now be shaded according to the light direction.<br><br>If you want to learn more lighting techniques, there are many more useful resources available.

## <div align='center'>Textures</div>

<br>In WebGL, textures aren't really considered as "images" per se. They can be interpreted as grids or arrays of vec4(or sometimes other types) values, called texels(pixels but in textures. "texel" isn't used commonly, as "pixel" can still be used almost always). Textures are mainly used for texturing, but can also be utilized to hold and transfer large amounts of data to the fragment shader.<br><br>We're going to look at the most simple and common usage for textures: applying images onto meshes.<br><br>To access a pixel in a texture, _"UV"_ coordinates(also called texture coordinates) are used. UV coordinates are vec2s that specify a pixel from a texture. They always range from 0 to 1, no matter the width or height of the texture. A _"u"_ coordinate is like the x value of a texture coordinate. A _"v"_ coordinate is like the y value of a texture coordinate, and goes up the texture as it increases.<br><br>For each pixel in a face, we need to figure out its appropriate UV value to find out what pixel in the texture it corresponds to. To do this, we specify a UV coordinate at each vertex of a face and use varyings in interpolate and transfer them into the fragment shader, where the texture's color at that point can be looked up and applied.<br><br>To start of, we add another attribute, a vec2, to each vertex. This will contain the UV coordinates at the corresponding corner of the texture.
```
let verts=[
    
	//front side
	-0.5,0.5,-0.5,  0,1,0,  0,0,-1,  1,0,
	-0.5,-0.5,-0.5,  0,1,0,  0,0,-1,  1,1,
	0.5,-0.5,-0.5,  0,1,0,  0,0,-1,  0,1,
	0.5,0.5,-0.5,  0,1,0,  0,0,-1,  0,0,
    
	//back side
	-0.5,0.5,0.5,  0,1,0,  0,0,1,  0,0,
	-0.5,-0.5,0.5,  0,1,0,  0,0,1,  0,1,
	0.5,-0.5,0.5,  0,1,0,  0,0,1,  1,1,
	0.5,0.5,0.5,  0,1,0,  0,0,1,  1,0,
    
	//top side
	-0.5,0.5,0.5,  0,1,0,  0,1,0,  1,0,
	-0.5,0.5,-0.5,  0,1,0,  0,1,0,  1,1,
	0.5,0.5,-0.5,  0,1,0,  0,1,0,  0,1,
	0.5,0.5,0.5,  0,1,0,  0,1,0,  0,0,
    
	//bottom side
	-0.5,-0.5,0.5,  0,1,0,  0,-1,0,  1,0,
	-0.5,-0.5,-0.5,  0,1,0,  0,-1,0,  1,1,
	0.5,-0.5,-0.5,  0,1,0,  0,-1,0,  0,1,
	0.5,-0.5,0.5,  0,1,0,  0,-1,0,  0,0,
    
	//left side
	-0.5,0.5,-0.5,  0,1,0,  -1,0,0,  0,0,
	-0.5,-0.5,-0.5,  0,1,0,  -1,0,0,  0,1,
	-0.5,-0.5,0.5,  0,1,0,  -1,0,0,  1,1,
	-0.5,0.5,0.5,  0,1,0,  -1,0,0,  1,0,
    
	//right side
	0.5,0.5,-0.5,  0,1,0,  1,0,0,  1,0,
	0.5,-0.5,-0.5,  0,1,0,  1,0,0,  1,1,
	0.5,-0.5,0.5,  0,1,0,  1,0,0,  0,1,
	0.5,0.5,0.5,  0,1,0,  1,0,0,  0,0,
]
```
The vertex UV attribute's location:
```
let vertUVLocation=gl.getAttribLocation(program,'vertUV')
gl.enableVertexAttribArray(vertUVLocation)
```
Update the attribute pointers:
```
//bytes per vertex. the total amount of values per a vertex(now it's 11(x,y,z,r,g,b,nx,ny,nz,u,v)) multiplied by 4(which is the amount of bytes in a float32)
let bpv=44

//3 values for the position, 0 bytes before the position values
gl.vertexAttribPointer(vertPosLocation,3,gl.FLOAT,gl.FALSE,bpv,0)

//3 values for the color, 3 values(x & y & z coords) * 4 bytes per value = 12 bytes before the color values
gl.vertexAttribPointer(vertColorLocation,3,gl.FLOAT,gl.FALSE,bpv,12)

//3 values for the normal, 6 values(x & y & z & nx & ny & nz coords) * 4 bytes per value = 24 bytes before the color values
gl.vertexAttribPointer(vertNormalLocation,3,gl.FLOAT,gl.FALSE,bpv,24)

//2 values for the uv, 9 values(x & y & z & nx & ny & nz coords, r & g & b values) * 4 bytes per value = 36 bytes before the color values
gl.vertexAttribPointer(vertUVLocation,2,gl.FLOAT,gl.FALSE,bpv,36)
```
We now create the texture to use. We need to know the width, height, and _"imageData"_ of the texture. An _"imageData"_ is just a giant _"Uint8Array"_ that has 4 color values(RGBA values) for each pixel. _"imageData"_ pixels go from left to right, and up to down. We'll keep the texture simple for now.
```
//creates a texture
let texture=gl.createTexture()

//binds the texture
gl.bindTexture(gl.TEXTURE_2D,texture)

//defines what's in the texture
//params: texture type, level(almost always at 0), format, width, height, border(almost always 0), internal format, type, imageData
gl.texImage2D(gl.TEXTURE_2D,0,gl.RGBA,2,2,0,gl.RGBA,gl.UNSIGNED_BYTE,new Uint8Array([0,0,0,0,0,0,0,100,0,0,0,100,0,0,0,0]))

//texture filtering: specifies how texels are picked when uvs are not directly on a texel.
//most common settings: nearest and linear
//nearest picks the closest texel to the specified uv point
//linear blends the closest texels get a smoother, non-pixely texture
gl.texParameteri(gl.TEXTURE_2D,gl.TEXTURE_MIN_FILTER,gl.NEAREST)
gl.texParameteri(gl.TEXTURE_2D,gl.TEXTURE_MAG_FILTER,gl.NEAREST)

//automatically creates several smaller versions of the texture to use when the rendered texture is smaller
//this will reduce pixely images and increase performance, but also will increase memory usage by 33.33%
gl.generateMipmap(gl.TEXTURE_2D)
```
And finally, add textures to the shaders. We input the texture as a _"uniform sampler2D"_.
```
let vertexShaderCode=`#version 300 es
    
    precision mediump float;
    
    in vec3 vertPos;
    in vec3 vertColor;
    in vec3 vertNormal;
    
    //the vertex uv attribute
    in vec2 vertUV;
    
    uniform mat4 modelMatrix;
    
    out vec3 pixColor;
    out vec3 pixNormal;
    
    //transfer and interpolate uvs using a varying
    out vec2 pixUV;
    
    void main(){
  	 
   	 pixColor=vertColor;
   	 pixNormal=vertNormal;
   	 pixUV=vertUV;
  	 
   	 gl_Position=modelMatrix*vec4(vertPos,1);
    }
`

let fragmentShaderCode=`#version 300 es
    
    precision mediump float;
    
    in vec3 pixColor;
    in vec3 pixNormal;
    
    //the vertex uv varying, interpolated and transferred from the vertex shader
    in vec2 pixUV;
    
    uniform sampler2D tex;
    
    out vec4 fragColor;
    
    void main(){
  	 
   	 vec3 lightDir=normalize(vec3(-0.7,-1.5,-1));
   	 float diffuse=dot(-lightDir,pixNormal)*0.5+0.5;
  	 
   	 //the surface color
   	 vec3 surfaceColor=pixColor;
  	 
   	 //the texture's texel color for this fragment
   	 //"texture" always outputs a vec4
   	 vec4 textureColor=texture(tex,pixUV);
  	 
   	 //applies texture's color based on the texel's alpha value
   	 //"mix" is linear interpolation and works with vectors
   	 surfaceColor=mix(surfaceColor,textureColor.rgb,textureColor.a);
  	 
   	 fragColor=vec4(surfaceColor*diffuse,1);
    }
`
```
You should see the same spinning cube, but this time, a 2x2 checkerboard pattern on the faces. Lighting is also applied after the texture.<br><br>In some programs, vertex colors won't be needed(replaced with UVs) as the textures themselves provide the colors. However, in this program, we use both to get more flexibility, which is fine for performance.<br><br>Also, notice that we use _"uniform sampler2D tex;"_ in the fragment shader, but we never uniformed the texture! WebGL automatically uniforms the currently bound texture at the time of the draw call. You can still uniform the texture yourself, it's optional:
```
let textureLocation=gl.getUniformLocation(program,'tex')

gl.uniform1i(textureLocation,texture)
```
When using WebGL1, usage of textures is much more limited. Texture sizes must be a power of 2 in order to generate mipmaps. WebGL2 gets rid of these terrible restrictions.

Now, let's spice up our texture! Loading images and turning them into textures is easy, as you only need the width, height, and data of the image.<br><br>Instead of loading images, we can also draw and generate a texture directly in our program. Because we only need the _"imageData"_, we can draw onto an invisible canvas and get its _"imageData"_ to use as our texture.<br><br>We'll do this with the simple and common _"Canvas2DRenderingContext"_. Start by adding an invisible canvas to draw an image on.
```
<canvas id='textureCanvas' width='256' height='256' style='display:none'></canvas>
```
Now we get the canvas and context:
```
//fetches the texture canvas
let texCanvas=document.getElementById('textureCanvas')

//the canvas2d api
let tex_ctx=texCanvas.getContext('2d')
```
Draw our texture!
```
tex_ctx.clearRect(0,0,texCanvas.width,texCanvas.height)

tex_ctx.fillStyle='black'
tex_ctx.font='bold 30px arial'

tex_ctx.fillText('this is a texture!',15,100)
tex_ctx.fillText('wowie :O',65,190)
```
We get the _"imageData"_ of the canvas using _"tex\_ctx.getImageData()"_. Updated _"gl.texImage2D()"_ function, with added texture width, height, and data:
```
gl.texImage2D(gl.TEXTURE_2D,0,gl.RGBA,texCanvas.width,texCanvas.height,0,gl.RGBA,gl.UNSIGNED_BYTE,tex_ctx.getImageData(0,0,texCanvas.width,texCanvas.height))
```
Our cube should now have a very cool texture! In the next section, we'll be learning about more features with textures.

## <div align='center'>More Texturing</div>

<br>In this section, we'll be learning about more ways to utilize textures to color meshes.<br><br>
#### <div align='center'>Multiple Textures</div>
You shouldn't use more than 1 texture in the fragment shader, but if you have to, here's how.<br><br>Start with the code from the **Textures** section. We need to create another texture:
```
let texture2=gl.createTexture()

gl.bindTexture(gl.TEXTURE_2D,texture2)

gl.texImage2D(gl.TEXTURE_2D,0,gl.RGBA,texCanvas.width,texCanvas.height,0,gl.RGBA,gl.UNSIGNED_BYTE,tex_ctx.getImageData(0,0,texCanvas.width,texCanvas.height))

gl.texParameteri(gl.TEXTURE_2D,gl.TEXTURE_MIN_FILTER,gl.NEAREST)
gl.texParameteri(gl.TEXTURE_2D,gl.TEXTURE_MAG_FILTER,gl.NEAREST)

gl.generateMipmap(gl.TEXTURE_2D)
```
To draw the second texture, we need to clear the texture canvas and draw before creating the second texture and after creating the first.
```
tex_ctx.clearRect(0,0,texCanvas.width,texCanvas.height)
tex_ctx.fillStyle='black'
tex_ctx.font='bold 30px arial'
tex_ctx.fillText('this is texture 1',15,100)

//create first texture here...

tex_ctx.clearRect(0,0,texCanvas.width,texCanvas.height)
tex_ctx.fillStyle='black'
tex_ctx.font='bold 30px arial'
tex_ctx.fillText('this is texture 2',15,190)

//create second texture here...
```
Now that we have our textures created, add another uniform into the fragment shader.
```
let fragmentShaderCode=`#version 300 es
    
	precision mediump float;
    
	in vec3 pixColor;
	in vec3 pixNormal;
	in vec2 pixUV;
    
	uniform sampler2D tex;
    
	//the second texture
	uniform sampler2D tex2;
    
	out vec4 fragColor;
    
	void main(){
   	 
    	vec3 lightDir=normalize(vec3(-0.7,-1.5,-1));
    	float diffuse=dot(-lightDir,pixNormal)*0.5+0.5;
   	 
    	vec3 surfaceColor=pixColor;
   	 
    	vec4 textureColor=texture(tex,pixUV);
   	 
    	//the 2nd texture's texel color
    	vec4 texture2Color=texture(tex2,pixUV);
   	 
    	surfaceColor=mix(surfaceColor,textureColor.rgb,textureColor.a);
   	 
    	//mix in the 2nd texture's color too
    	surfaceColor=mix(surfaceColor,textureColor.rgb,texture2Color.a);
   	 
    	fragColor=vec4(surfaceColor*diffuse,1);
	}
`
```
To uniform the 2 textures, you need to use the _"gl.activeTextures()"_ function.
```
//gets locations
let textureLocation=gl.getUniformLocation(program,'tex')
let texture2Location=gl.getUniformLocation(program,'tex2')

//"texture" is now known as active texture "0"
gl.activeTexture(gl.TEXTURE0)
gl.bindTexture(gl.TEXTURE_2D,texture)

//"texture2" is now known as active texture "1"
gl.activeTexture(gl.TEXTURE1)
gl.bindTexture(gl.TEXTURE_2D,texture2)

//uniform based on the texture's active number value
gl.uniform1i(textureLocation,0)
gl.uniform1i(texture2Location,1)
```
You should see that the faces of the cube contain 2 lines of text, each from their own textures.<br><br>
#### <div align='center'>Texture Atlas</div>
A texture atlas is a large texture with multiple other textures placed side by side. In programs that require many different textures(such as Minecraft), this is the only solution.<br><br>Start with the code from the **Textures** section. We are going to change the texture and draw 6 different textures next to each other.
```
tex_ctx.clearRect(0,0,texCanvas.width,texCanvas.height)

tex_ctx.fillStyle='black'
tex_ctx.lineWidth=5
tex_ctx.strokeStyle='black'
tex_ctx.font='bold 40px arial'

tex_ctx.strokeRect(0,0,256/3,256/3)
tex_ctx.strokeRect(256/3,0,256/3,256/3)
tex_ctx.strokeRect(256*2/3,0,256/3,256/3)
tex_ctx.strokeRect(2,256/3,256/3,256/3)
tex_ctx.strokeRect(256/3,256/3,256/3,256/3)
tex_ctx.strokeRect(256*2/3,256/3,256/3,256/3)

tex_ctx.fillText('A',30,55)
tex_ctx.fillText('B',30+85,55)
tex_ctx.fillText('C',30+85*2,55)
tex_ctx.fillText('D',30,55+85)
tex_ctx.fillText('E',30+85,55+85)
tex_ctx.fillText('F',30+85*2,55+85)
```
Adjust the texture coordinates to cover a different portion of the texture for each face:
```
let s=0.333333

let verts=[
    
	//front side
	-0.5,0.5,-0.5,  0,1,0,  0,0,-1,  s,s,
	-0.5,-0.5,-0.5,  0,1,0,  0,0,-1,  s,s+s,
	0.5,-0.5,-0.5,  0,1,0,  0,0,-1,  0,s+s,
	0.5,0.5,-0.5,  0,1,0,  0,0,-1,  0,s,
    
	//back side
	-0.5,0.5,0.5,  0,1,0,  0,0,1,  s,0,
	-0.5,-0.5,0.5,  0,1,0,  0,0,1,  s,s,
	0.5,-0.5,0.5,  0,1,0,  0,0,1,  s+s,s,
	0.5,0.5,0.5,  0,1,0,  0,0,1,  s+s,0,
    
	//top side
	-0.5,0.5,0.5,  0,1,0,  0,1,0,  s,s,
	-0.5,0.5,-0.5,  0,1,0,  0,1,0,  s,s+s,
	0.5,0.5,-0.5,  0,1,0,  0,1,0,  s+s,s+s,
	0.5,0.5,0.5,  0,1,0,  0,1,0,  s+s,s,
    
	//bottom side
	-0.5,-0.5,0.5,  0,1,0,  0,-1,0,  s+s+s,s,
	-0.5,-0.5,-0.5,  0,1,0,  0,-1,0,  s+s+s,s+s,
	0.5,-0.5,-0.5,  0,1,0,  0,-1,0,  s+s,s+s,
	0.5,-0.5,0.5,  0,1,0,  0,-1,0,  s+s,s,
    
	//left side
	-0.5,0.5,-0.5,  0,1,0,  -1,0,0,  s,0,
	-0.5,-0.5,-0.5,  0,1,0,  -1,0,0,  s,s,
	-0.5,-0.5,0.5,  0,1,0,  -1,0,0,  0,s,
	-0.5,0.5,0.5,  0,1,0,  -1,0,0,  0,0,
    
	//right side
	0.5,0.5,-0.5,  0,1,0,  1,0,0,  s+s+s,0,
	0.5,-0.5,-0.5,  0,1,0,  1,0,0,  s+s+s,s,
	0.5,-0.5,0.5,  0,1,0,  1,0,0,  s+s,s,
	0.5,0.5,0.5,  0,1,0,  1,0,0,  s+s,0,
]
```
You should now see a different texture for each face! This method is extremely efficient and is the only viable method for using multiple textures at a fast speed.

## <div align='center'>Framebuffers & Post Processing</div>

<br>In WebGL, framebuffers are like collections of attachments of different types of textures. The attachments can be color, depth, or stencil attachments. Framebuffers are especially useful as they allow you to render to a texture. You can attach a texture to a framebuffer, bind the framebuffer, draw geometry, and it'll end up in the framebuffer's attached texture instead of the canvas!<br><br>Rendering to a texture will allow you to perform post processing on the rendered image. You can apply effects like blurring, bloom, and color grading to the final image.<br><br>First, create a new _"WebGLProgramObject"_ that performs the post processing. In this example, the post processing shader will invert the colors of the rendered image.
```
let pp_program=createProgram(`#version 300 es
    
	precision mediump float;
    
	in vec2 vertPos;
    
	out vec2 pixUV;
    
	void main(){
   	 
    	pixUV=vertPos*0.5+0.5;
    	gl_Position=vec4(vertPos,0,1);
	}

`,`#version 300 es
    
	precision mediump float;
    
	in vec2 pixUV;
    
	uniform sampler2D tex;
    
	out vec4 fragColor;
    
	void main(){
   	 
    	fragColor=vec4(1.0-texture(tex,pixUV).rgb,1);
	}
`)
```
After creating the original mesh, we create a 2D mesh that covers the screen. The screen mesh can be created using 2 triangles forming a rectangle that covers the screen, or from 1 large triangle. Here, we also get the attribute location while we're at it:
```
let pp_vertPosLocation=gl.getAttribLocation(program,'vertPos')
gl.enableVertexAttribArray(pp_vertPosLocation)

let pp_buffer=gl.createBuffer()

gl.bindBuffer(gl.ARRAY_BUFFER,pp_buffer)

//vertices are a large triangle covering the screen
//the format of the buffer is x1,y1,x2,y2,x3,y3
gl.bufferData(gl.ARRAY_BUFFER,new Float32Array([-1,-1,3,-1,-1,3]),gl.STATIC_DRAW)
```
We now create the texture and framebuffer. We also have to attach a depth renderbuffer to the framebuffer in order for the program to perform depth testing when rendering to a texture.
```
//create the texture to be rendered to
let texture=gl.createTexture()
gl.bindTexture(gl.TEXTURE_2D,texture)

//data is "null" as it's not needed to be set here
gl.texImage2D(gl.TEXTURE_2D,0,gl.RGBA,canvas.width,canvas.height,0,gl.RGBA,gl.UNSIGNED_BYTE,null)

gl.texParameteri(gl.TEXTURE_2D,gl.TEXTURE_MIN_FILTER,gl.LINEAR)


//create the framebuffer
let framebuffer=gl.createFramebuffer()

//bind the framebuffer
gl.bindFramebuffer(gl.FRAMEBUFFER,framebuffer)

//attach the texture to the framebuffer
gl.framebufferTexture2D(gl.FRAMEBUFFER,gl.COLOR_ATTACHMENT0,gl.TEXTURE_2D,texture,0)

//create a depth renderbuffer and attach it to the framebuffer
let depthBuffer=gl.createRenderbuffer()
gl.bindRenderbuffer(gl.RENDERBUFFER,depthBuffer)
gl.renderbufferStorage(gl.RENDERBUFFER,gl.DEPTH_COMPONENT16,canvas.width,canvas.height)
gl.framebufferRenderbuffer(gl.FRAMEBUFFER,gl.DEPTH_ATTACHMENT,gl.RENDERBUFFER,depthBuffer)
```
Then, we update the rendering process...
```
//stuff rendered after a bound framebuffer with render to the framebuffer's attached texture
gl.bindFramebuffer(gl.FRAMEBUFFER,framebuffer)

//use the normal program
gl.useProgram(program)


//original cube mesh rendering goes here...


//unbinds the framebuffer, meaning stuff after draws to the real canvas
gl.bindFramebuffer(gl.FRAMEBUFFER,null)

//"texture" now contains the rendered cube!
gl.bindTexture(gl.TEXTURE_2D,texture)

//use the post processing program
gl.useProgram(pp_program)

//draw the big triangle that covers the screen
gl.bindBuffer(gl.ARRAY_BUFFER,pp_buffer)
gl.vertexAttribPointer(pp_vertPosLocation,2,gl.FLOAT,gl.FALSE,8,0)
gl.drawArrays(gl.TRIANGLES,0,3)
```
You should see the same cube, but the final rendered image's color is inverted!







<br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br><br>







