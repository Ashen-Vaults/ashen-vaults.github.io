---
title: "Perceptron Machine Learning"
layout: post
date: 2018-02-02 04:55
unity_dir: perceptron_master
tag: 
- unity3d
- c#
- machine learning
- data science
- ai

image: 
headerImage: false
projects: true
hidden: false # don't count this post in blog pagination
description: "A Unity3D implementation of a simple machine learning algorithm for a classification problem"
category: project
author: alexsh
externalLink: false
---

## About:

This is a supervised machine learning algorithm which classifies a point based on where it falls relative to a line. Supervised learning works by giving the machine data that we know is correct so that it can use it to learn from. This is different from unsupervised learning, where no training data is given, and the machine has to discover 
patterns from the data on it's own.


First we have to create our line. we place two points on opposite sides and connect them. To facilitate with calculating the sides of the visible area, I created a static class which acts as a dispatch table, where a string, ("bottom_left"), points to a function that returns a Vector3. This way if the screen or camera are ever resized and we are running the simulation, everything can recalculate during runtime. 

>  
	public static class LineDispatch  
	{
        public static Dictionary<string,Func<Vector3>> lineCoordinates = new  Dictionary<string,Func<Vector3>>()
        {
            { "bottom_left", ()=> { return Camera.main.ScreenToWorldPoint(new Vector3(0, 0, Camera.main.farClipPlane/2)); }},
            { "bottom_right", ()=> { return Camera.main.ScreenToWorldPoint(new Vector3(Screen.width, 0, Camera.main.farClipPlane/2)); }},
            { "top_left", ()=> { return Camera.main.ScreenToWorldPoint(new Vector3(0, Screen.height, Camera.main.farClipPlane/2)); }},
            { "top_right", ()=> { return Camera.main.ScreenToWorldPoint(new Vector3(Screen.width, Screen.height, Camera.main.farClipPlane/2)); }},
            { "center", ()=> { return Camera.main.ScreenToWorldPoint(new Vector3(Screen.width/2, Screen.height/2, Camera.main.farClipPlane/2)); }}
        };
    }


This way if I want a specific point within the visible screen, I can just ask for it, and let the calculation happen in a black box, without worrying about re-implementing it. 

> 
	void DrawLines()
	{
	    _lines.ForEach(l => l.DrawLine( LineDispatch.lineCoordinates["bottom_left"](), LineDispatch.lineCoordinates["top_right"]() ));
	    DrawPoints(() => { return _lines.FirstOrDefault(l => l.data._type == "Target");});
	}

We then create an arbitrary number of points within our Cartesian plane. As each point is instantiated, it checks to see which side of the line it falls on by taking the perpendicular dot product:

>    
	public int GetSide(Vector3 point)
	{
	    Vector3 diff = GetEndPoint() - GetStartPoint();
	    Vector3 perp = new Vector3(-diff.y, diff.x,0);
	    float d = Vector3.Dot(point - GetStartPoint(), perp);
	    return Math.Sign(d);
	}



>
	IEnumerator AnimateSimulation(float duration=0.5f)
	{
	    for (int i = 0; i < _points.Count; i++)
	    { 
	        _perceptron.Train(_inputs, _points[i].Label);
	        int guess = _perceptron.Guess(_inputs);
	        _points[i].Compare(guess);
	        yield return new WaitForSeconds(duration);
	    }
	}



### Weights:
Each input that we feed into the perceptron is weighted by some value. All of the weights are then multiplied by their inputs, which gives us the sum of all the data.

These points will be used as our training data for the perceptron. 
When the simulation is run, the perceptron uses every points label as training. 
It first attempts a guess, which we then use to calculate the standard error
from the target value. It will then re-adjust the weights it's using gradient descent until it reaches it's optimal value and the machine can solve the classification issue.


### Gradient Descent:
An optimization technique which minimizes the standard error of a model. 



>
	public void Train(List<float> inputs, int target)
	{
	    int guess = Guess(inputs);
	    int error = target - guess;
	    for (int i = 0; i < _weights.Count; i++)
	    {
	        _weights[i] += error * inputs[i] * _learningRate;    
	    }
	}



Whenever a guess is made, the perceptron is given a collection of inputs,
which it then uses to tweak each of it's weights.
> 
	public int Guess(List<float> inputs)
	{
    float sum = 0;
    for (int i = 0; i < _weights.Count; i++)
    {
        sum += inputs[i] * _weights[i];   
    }
    return Math.Sign(sum);
	}

### Activation Function
Conforms the output to a desired range. What happens when the data comes
out of the neuron. Given any number, convert it to either +1 or -1. In this case 
I am using the Math.Sign function. Feed forward.
#What to do with zero?

