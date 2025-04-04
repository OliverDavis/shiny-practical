# Getting started with Shiny

Interactive graphics can help us handle complexity by allowing users to interact with the data. Here we're going to explore a popular way of doing this in R, using the Shiny package.

## Intended learning outcomes
After this practical, you should be able to:
* explain the principles of the Shiny package
* use Shiny to make interactive data graphics

## Getting started

In this session, we’ll be using the R package Shiny to build an interactive web app. Although using Shiny means you don’t have to worry about coding HTML, Javascript or CSS (all the code is R), there are some concepts common to other web apps that you’ll need to appreciate what’s going on. We’ll introduce those as they appear below. First of all, create a new Shiny app in R Studio:

```File -> New File -> Shiny Web App …```

Choose a name for the app, and where you’d like it to live. You’ll see that R Studio has helped us by giving us the boilerplate code necessary to get a Shiny app running, and populates it with data on the eruption of the Old Faithful Geyser in Yellowstone Park, USA. Before we get into how the code works, hit the “Run App” button at the top of the screen and play around with the demo app.

## The structure of a Shiny app

You’ll see that Shiny has generated a web page with two sections: a histogram, and a slider to adjust the number of bins in the histogram. This is the User Interface, or UI, often called the front end. If you look at the R Studio console, you’ll notice that you can’t enter anything because it’s currently running an R server that’s listening out for your inputs on the front end slider and responding by updating the graph; this server is often called the back end. When you write a Shiny app, you’ll specify:

* how you want the UI to appear and what levers you want to give the user to play with the data
* what processing you’d like the server to do
* how the two should talk to each other

Let’s go back to the R code now. After loading the Shiny package, you’ll see that there are two main sections to the code: a `ui` object, and a `server` object, which correspond to the two main parts of a web app discussed above. The last line of the code runs the application.

Looking more closely at the `ui` object, you’ll see that the code is to do with the layout of the page and the user inputs. Within the `fluidPage`, there’s a `titlePanel`, followed by a `sidebarLayout` that contains a `sidebarPanel` with the user input slider, then a `mainPanel` that contains the `plotOutput`, in this case a histogram. Note that both the `sliderInput` and the `plotOutput` have names: `bins` and `distPlot`.

Looking at the server object now, you’ll see it’s a function with the arguments input and output. The output area is referred to by the name given to it in `plotOutput` above: `output$distPlot`. The `renderPlot` function tells Shiny what to put in the output area defined in the `ui` object, using the `hist` function we met in the static data vis practical earlier. You can see that the user input from the `sliderInput` named “bins” (`input$bins`) is used to calculate bins variable that is passed to the “breaks” argument in the `hist` function.

The last thing to notice is that the `server` function is not actually called in the code; instead, it is passed as an argument to the `shinyApp` function in the last line. Unlike the usual R paradigm of imperative programming where you issue a command and expect it to be carried out immediately, Shiny adopts a declarative programming paradigm where you describe the  goals and constraints, and rely on Shiny to decide when to actually execute the code.

## Reactivity

But when does Shiny choose to execute the code? Well, you’ll have noticed that the histogram updates every time you move the slider; it reacts to user input. This is a principle called “reactivity” that is common in web apps: Shiny chooses to recalculate the outputs whenever the input changes. This means that any calculations you do based on user input must either be contained by an output function like renderPlot, or be wrapped up in a reactive expression using `reactive()` and assigned to a variable. You can then use the reactive expression by calling it like a function.

## Find out more

That should give you an idea of the main features of Shiny. Of course, there’s much more to learn. A good place to look is Hadley Wickham’s [Mastering Shiny](https://mastering-shiny.org/index.html) book, particularly [Chapter 2](https://mastering-shiny.org/basic-ui.html), which runs through the main UI components that you’ll need for building apps, and [Chapter 4](https://mastering-shiny.org/basic-case-study.html) that provides an example of a more complex Shiny app using medical data. As usual, Posit provide a very handy [cheat sheet](https://rstudio.github.io/cheatsheets/html/shiny.html).

## Challenge 1 (10 mins)
Using the `medicaldata` package, modify the basic Shiny app to take user inputs and produce a plot of one of the medical data sets using `ggplot2`. Add a table of the first few lines of data.

<details>
<summary>Solution</summary>

```R
library(shiny)
library(medicaldata)
library(ggplot2)

# Define UI for application that draws a histogram
ui <- fluidPage(

    # Application title
    titlePanel("Baseline polyp count in males and females"),

    # Flexible fluid row with narrow column for slider input for number of bins 
    fluidRow(
        column(width=4,
            sliderInput("bins",
                        "Number of bins:",
                        min = 1,
                        max = 50,
                        value = 30)
        ),

        # Show a plot of the generated distribution
        column(width=8,
           plotOutput("distPlot")
        )
    ),
    
    # A new row for the data table
    fluidRow(
      column(width=12,
             DT::DTOutput("dat_table")
      )
    )
)

# Define server logic required to draw a histogram
server <- function(input, output) {

    # Render the plot
    output$distPlot <- renderPlot({
      
      ggplot(polyps, aes(x=baseline, colour=sex, fill=sex)) +
        geom_histogram(alpha=0.5, position="identity", bins=input$bins) +
        facet_grid(sex ~ .)
      
    })
    
    # Render the table
    output$dat_table <- DT::renderDT(polyps, options=list(pageLength=10))
}

# Run the application 
shinyApp(ui = ui, server = server)

```

</details>

## Challenge 2 (60 mins)
Using this [simulated data set](moons.csv), can you use your knowledge of [KNN classification](https://www.rdocumentation.org/packages/class/versions/7.3-23/topics/knn) and `ggplot2` to reproduce this plot of a KNN decision boundary?

![plot of a KNN decision boundary](knn.png)

<details>
<summary>Solution</summary>

```R

library(readr)
library(ggplot2)
library(dplyr)
library(class)

train <- read_csv("moons.csv") |>
  mutate(cl=as.factor(cl))

# Define the grid limits
x_min <- min(train$x) - 0.2
x_max <- max(train$x) + 0.2
y_min <- min(train$y) - 0.2
y_max <- max(train$y) + 0.2

# Create a grid of values
test <- expand.grid(x = seq(x_min, x_max, by = 0.1),
                    y = seq(y_min, y_max, by = 0.1))


# Train the k-NN model and predict on the grid
k <- 20  # Number of neighbors
classif <- knn(train = train[,1:2], prob = TRUE, test = test, 
                         cl = train$cl, k = k)

prob <- attr(classif, "prob")

dataf <- bind_rows(mutate(test,
                          prob=prob,
                          cls=0,
                          prob_cls=ifelse(classif==cls,
                                          1, 0)),
                   mutate(test,
                          prob=prob,
                          cls=1,
                          prob_cls=ifelse(classif==cls,
                                          1, 0))) |>
  mutate(cls=as.factor(cls))


ggplot(dataf) +
  geom_point(aes(x=x, y=y, col=cls, size=prob, alpha=0.2),
             data = mutate(test, cls=classif),
             show.legend=c(alpha=FALSE, cls=TRUE, prob=TRUE)) + 
  scale_size(range=c(0,1.5)) +
  geom_contour(aes(x=x, y=y, z=prob_cls, group=cls, color=cls),
               bins=2,
               data=dataf) +
  geom_point(aes(x=x, y=y, col=cl),
             size=2,
             data=train) +
  geom_point(aes(x=x, y=y),
             size=2, shape=1,
             data=train) +
  labs(title = "KNN decision boundary") +
  guides(colour=guide_legend("class"),
         size=guide_legend("probability")) +
  theme_minimal()

```

</details>


## Challenge 3 (20 mins)
Now use the code from Challenge 2 to make a Shiny app that shows how the decision boundary changes with different values of k.

<details>
<summary>Solution</summary>

```R

library(readr)
library(ggplot2)
library(dplyr)
library(class)
library(markdown)

train <- read_csv("moons.csv") |>
  mutate(cl=as.factor(cl))

# Define the grid limits
x_min <- min(train$x) - 0.2
x_max <- max(train$x) + 0.2
y_min <- min(train$y) - 0.2
y_max <- max(train$y) + 0.2

# Create a grid of values
test <- expand.grid(x = seq(x_min, x_max, by = 0.1),
                    y = seq(y_min, y_max, by = 0.1))

# Define UI for application that draws a histogram
ui <- fluidPage(

    # Application title
    titlePanel("Value of k in KNN"),

    # Flexible fluid row with narrow column for slider input for value of k
    fluidRow(
      column(width=8,
             includeMarkdown("knn.md")
      ),  
      
      column(width=4,
            sliderInput("k",
                        "Value of k:",
                        min = 1,
                        max = 100,
                        value = 5)
      )

    ),
    
    fluidRow(
      column(width=12,
             plotOutput("decision_plot", height="500px")
      )
    )
    
)


# Define server logic required to draw a histogram
server <- function(input, output) {
  
  # Render the plot
  output$decision_plot <- renderPlot({
    
    classif <- knn(train = train[,1:2], prob = TRUE, test = test, 
                   cl = train$cl, k = input$k)
    
    prob <- attr(classif, "prob")
    
    dataf <- bind_rows(mutate(test,
                              prob=prob,
                              cls=0,
                              prob_cls=ifelse(classif==cls,
                                              1, 0)),
                       mutate(test,
                              prob=prob,
                              cls=1,
                              prob_cls=ifelse(classif==cls,
                                              1, 0))) |>
      mutate(cls=as.factor(cls))
    
    ggplot(dataf) +
      coord_fixed() +
      geom_point(aes(x=x, y=y, col=cls, size=prob, alpha=0.2),
                 data = mutate(test, cls=classif),
                 show.legend=c(alpha=FALSE, cls=TRUE, prob=TRUE)) + 
      scale_size(range=c(0,2)) +
      geom_contour(aes(x=x, y=y, z=prob_cls, group=cls, color=cls),
                   bins=2,
                   data=dataf) +
      geom_point(aes(x=x, y=y, col=cl),
                 size=3,
                 data=train) +
      geom_point(aes(x=x, y=y),
                 size=3, shape=1,
                 data=train) +
      guides(colour=guide_legend("class"),
             size=guide_legend("probability")) +
      theme_minimal()
    
  })
    
}

# Run the application 
shinyApp(ui = ui, server = server)

```

</details>

## Challenge 4 (60 mins)
Shiny is now also available as a [Python package](https://shiny.posit.co/py/). How would you reimplement one of the visualisations you've made in R using Python?

<details>
<summary>Solution part one (static plot)</summary>

```python

import pandas as pd
import numpy as np
from sklearn.neighbors import KNeighborsClassifier
from plotnine import ggplot, aes, geom_point, scale_size, guides, guide_legend, theme_tufte
import matplotlib.pyplot as plt

# Load the dataset
train = pd.read_csv('./moons.csv')
train.cl = train.cl.astype('category')

# Find the min and max values for x and y to create a grid for test
x_min = train['x'].min() - 0.2
x_max = train['x'].max() + 0.2
y_min = train['y'].min() - 0.2
y_max = train['y'].max() + 0.2

test = pd.DataFrame([(x,y) for x in np.arange(x_min, x_max + 0.1, 0.1) for y in np.arange(y_min, y_max + 0.1, 0.1)],
                    columns=['x', 'y'])

# Set the value of k for KNN and fit the model
k = 20

knn = KNeighborsClassifier(n_neighbors=k, weights='distance', p=2)
knn = knn.fit(train[['x','y']], train['cl'])

# Now predict classfication and probabilities across the grid
classif = knn.predict(test)
prob = np.array([max(p) for p in knn.predict_proba(test)])

test['cls'] = classif
test.cls = test.cls.astype('category')
test['prob'] = prob

# Use plotnine (a python implementation of ggplot) to plot the results
# Note that geom_contour is not available in plotnine
knn_plot = (
    ggplot(test)
    + geom_point(aes(x='x', y='y', colour='cls', size='prob', alpha=0.2), stroke=0, show_legend={'alpha': False})
    + scale_size(range=[0, 2.25])
    + geom_point(aes(x='x', y='y', colour='cl'), size=3, stroke=0, data=train)
    + geom_point(aes(x='x', y='y'), size=3, fill='none', stroke=0.3, data=train)
    + guides(colour=guide_legend('class'), size=guide_legend('probability'))
    + theme_tufte()
)

# Save the plot
knn_plot.save('knn_plot.png', width=6, height=4, dpi=300)

```

![plot of a KNN decision boundary using python](knn_plot.png)

</details>

<details>
<summary>Solution part two (interactive plot)</summary>

```python

import pandas as pd
import numpy as np
from sklearn.neighbors import KNeighborsClassifier
from plotnine import ggplot, aes, geom_point, scale_size, guides, guide_legend, theme_tufte
import matplotlib.pyplot as plt

from shiny import App, render, ui

# Load the dataset
train = pd.read_csv('./moons.csv')
train.cl = train.cl.astype('category')

# Find the min and max values for x and y to create a grid for test
x_min = train['x'].min() - 0.2
x_max = train['x'].max() + 0.2
y_min = train['y'].min() - 0.2
y_max = train['y'].max() + 0.2

test = pd.DataFrame([(x,y) for x in np.arange(x_min, x_max + 0.1, 0.1) for y in np.arange(y_min, y_max + 0.1, 0.1)],
                    columns=['x', 'y'])


app_ui = ui.page_fluid(
    ui.panel_title("Value of k in KNN"),
    ui.row(
        ui.column(
            8,
            ui.markdown(
                """
                Changing the value of k can influence the decision boundary in k-nearest-neighbour classification.
                Use the slider to change the value of k and watch how it affects the classification of points in this simulated data set.
                """
            ),
        ),
        ui.column(
            4,
            ui.input_slider(
                "k",
                "Value of k",
                min=1,
                max=100,
                value=5,
                step=1,
                width="100%",
            )
        )
    ),
    ui.row(
        ui.column(
            12,
            ui.output_plot("decision_plot", width="100%", height="500px"),
        )
    )
)

def server(input, output, session):
    @render.plot
    def decision_plot():
                
        # Set the value of k for KNN and fit the model
        k = input.k()

        knn = KNeighborsClassifier(n_neighbors=k,  p=2)
        knn = knn.fit(train[['x','y']], train['cl'])

        # Now predict classfication and probabilities across the grid
        classif = knn.predict(test)
        prob = np.array([max(p) for p in knn.predict_proba(test)])

        out = test.copy()

        out['cls'] = classif
        out.cls = out.cls.astype('category')
        out['prob'] = prob

        # Use plotnine (a python implementation of ggplot) to plot the results
        # Note that geom_contour is not available in plotnine
        knn_plot = (
            ggplot(out)
            + geom_point(aes(x='x', y='y', colour='cls', size='prob', alpha=0.2), stroke=0, show_legend={'alpha': False})
            + scale_size(range=[0.1, 2.25])
            + geom_point(aes(x='x', y='y', colour='cl'), size=3, stroke=0, data=train)
            + geom_point(aes(x='x', y='y'), size=3, fill='none', stroke=0.3, data=train)
            + guides(colour=guide_legend('class'), size=guide_legend('probability'))
            + theme_tufte()
        )

        return knn_plot

app = App(app_ui, server)

```

</details>
