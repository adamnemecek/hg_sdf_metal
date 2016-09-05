    ////////////////////////////////////////////////////////////////  
    //  
    //                           HG_SDF  
    //  
    //     GLSL LIBRARY FOR BUILDING SIGNED DISTANCE BOUNDS  
    //  
    //     version 2016-01-10  
    //     Metal version 2016-09-05 (modifications by @warrenm)  
    //  
    //     Check http://mercury.sexy/hg_sdf for updates  
    //     and usage examples. Send feedback to spheretracing@mercury.sexy.  
    //  
    //     Brought to you by MERCURY http://mercury.sexy  
    //  
    //  
    //  
    // Released as Creative Commons Attribution-NonCommercial (CC BY-NC)  
    //  
    ////////////////////////////////////////////////////////////////  
    //  
    // How to use this:  
    //  
    // 1. Build some system to #include glsl files in each other.  
    //   Include this one at the very start. Or just paste everywhere.  
    // 2. Build a sphere tracer. See those papers:  
    //   * "Sphere Tracing" http://graphics.cs.illinois.edu/sites/default/files/zeno.pdf  
    //   * "Enhanced Sphere Tracing" http://lgdv.cs.fau.de/get/2234  
    //   The Raymnarching Toolbox Thread on pouet can be helpful as well  
    //   http://www.pouet.net/topic.php?which=7931&page=1  
    //   and contains links to many more resources.  
    // 3. Use the tools in this library to build your distance bound f().  
    // 4. ???  
    // 5. Win a compo.  
    //  
    // (6. Buy us a beer or a good vodka or something, if you like.)  
    //  
    ////////////////////////////////////////////////////////////////  
    //  
    // Table of Contents:  
    //  
    // * Helper functions and macros  
    // * Collection of some primitive objects  
    // * Domain Manipulation operators  
    // * Object combination operators  
    //  
    ////////////////////////////////////////////////////////////////  
    //  
    // Why use this?  
    //  
    // The point of this lib is that everything is structured according  
    // to patterns that we ended up using when building geometry.  
    // It makes it more easy to write code that is reusable and that somebody  
    // else can actually understand. Especially code on Shadertoy (which seems  
    // to be what everybody else is looking at for "inspiration") tends to be  
    // really ugly. So we were forced to do something about the situation and  
    // release this lib ;)  
    //  
    // Everything in here can probably be done in some better way.  
    // Please experiment. We'd love some feedback, especially if you  
    // use it in a scene production.  
    //  
    // The main patterns for building geometry this way are:  
    // * Stay Lipschitz continuous. That means: don't have any distance  
    //   gradient larger than 1. Try to be as close to 1 as possible -  
    //   Distances are euclidean distances, don't fudge around.  
    //   Underestimating distances will happen. That's why calling  
    //   it a "distance bound" is more correct. Don't ever multiply  
    //   distances by some value to "fix" a Lipschitz continuity  
    //   violation. The invariant is: each fSomething() function returns  
    //   a correct distance bound.  
    // * Use very few primitives and combine them as building blocks  
    //   using combine opertors that preserve the invariant.  
    // * Multiply objects by repeating the domain (space).  
    //   If you are using a loop inside your distance function, you are  
    //   probably doing it wrong (or you are building boring fractals).  
    // * At right-angle intersections between objects, build a new local  
    //   coordinate system from the two distances to combine them in  
    //   interesting ways.  
    // * As usual, there are always times when it is best to not follow  
    //   specific patterns.  
    //  
    ////////////////////////////////////////////////////////////////  
    //  
    // FAQ  
    //  
    // Q: Why is there no sphere tracing code in this lib?  
    // A: Because our system is way too complex and always changing.  
    //    This is the constant part. Also we'd like everyone to  
    //    explore for themselves.  
    //  
    // Q: This does not work when I paste it into Shadertoy!!!!  
    // A: Yes. It is GLSL, not GLSL ES. We like real OpenGL  
    //    because it has way more features and is more likely  
    //    to work compared to browser-based WebGL. We recommend  
    //    you consider using OpenGL for your productions. Most  
    //    of this can be ported easily though.  
    //  
    // Q: How do I material?  
    // A: We recommend something like this:  
    //    Write a material ID, the distance and the local coordinate  
    //    p into some global variables whenever an object's distance is  
    //    smaller than the stored distance. Then, at the end, evaluate  
    //    the material to get color, roughness, etc., and do the shading.  
    //  
    // Q: I found an error. Or I made some function that would fit in  
    //    in this lib. Or I have some suggestion.  
    // A: Awesome! Drop us a mail at spheretracing@mercury.sexy.  
    //  
    // Q: Why is this not on github?  
    // A: Because we were too lazy. If we get bugged about it enough,  
    //    we'll do it.  
    //  
    // Q: Your license sucks for me.  
    // A: Oh. What should we change it to?  
    //  
    // Q: I have trouble understanding what is going on with my distances.  
    // A: Some visualization of the distance field helps. Try drawing a  
    //    plane that you can sweep through your scene with some color  
    //    representation of the distance field at each point and/or iso  
    //    lines at regular intervals. Visualizing the length of the  
    //    gradient (or better: how much it deviates from being equal to 1)  
    //    is immensely helpful for understanding which parts of the  
    //    distance field are broken.  
    //  
    ////////////////////////////////////////////////////////////////  
