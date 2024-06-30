# Experiments with Claude Sonnet 3.5/Google Gemini

Inspried from [this](https://x.com/minchoi/status/1804928914478993816) tweet I decided to give a spin to Claude Sonnet 3.5.

I used the same prompt as mentioned in the tweet and chose the same Q1 2024 Tesla report for generating quick analysis with the help of Claude Sonnet 3.5.

My takeaways
- The model is smart enough to capture top line, bottom line etc. as performance metrics but what surprised me more is since it is a vehicle manufacturing company it gave importance to number of vehicles delivered/produced. I would want to try the same analysis for a company from other vertical and see if the model can really pick up metrics that matter to that industry
- When prompted about creating a dashboard, at first it gave an option to create such reports using tools like Tabelau, Qlik, Power BI etc. but when reminded that it can help generate code, gave a python code with plotly as a visualization library
- It kept on insisting me to run the code and refused to share the output of the execution by itself, which is fine
- The code generated was pretty good and almost worked in the first attempt
- I copy pasted the code in Google Colab and used Gemini to correct
- Google is aggresively pushing Gemini in all their product line and the experience is quite decent 
- Though while explaing and correcting the code in Google Colab it didn't complete the code in its entirety,  to me it was a let down
- Then I asked Claude itself to correct the code and it did a fabulous job. I just prompted that there is an error in the code, can you correct it and not only it corrected the code but also explained what the correction was, pretty impressive. I think the hype around Sonnet being better at coding may be right, need to test more

This was a theoratical exercise, nonetheless what this means is anyone with access to these models can pretty much do anything at a speed of light if they know what they want to do, here it was a financial analysis of a company's quarterly report. Beware research analysts :) 

I decided to use Claude for two practical examples related to my recent work

Use case 1: I had a data file which had certain columns which I wanted to pivot down. While I could do it in Excel myself pretty much easily, decided to throw the problem to Claude with the help of python and to my surprise it created a perfect code in the first attempt. Takeaway for me was if you can convert your ask in a precise prompt, these model can work magic

Use case 2: I had a schema of the data for which I wanted to create synthetic data. Again prompting is an art I believe which will be valuable in the era of AI. But the beauty of the model is, there were sample values and description for all the columns for which I wanted to generate synthetic data and model extrapolated that knowledge to create really good synthetic data, it just didn't decide to generate any random value but actually generated picture perfect synthetic data using the inherent knowledge it might have. e.g. Specialty of the doctor, lab tests conducted on patients, medicine names generated were to the point and accurate 


I have decided to use these models for my day to day work and will share any new findings that I come across but the future looks pretty exciting with these models to say the least.


