# Streamlit

## Collaborative EDA Made Simple

**Posted by Joe Blankenship on November 23, 2021**

## Introduction

For most data analysts, data engineers, and data scientists using Python, a simple import and few commands in IPython or Jupyter can help quickly characterize the data. This characterization is often very concise and technical which is great for enabling technical team members to communicate on data processing and delivery tasks.

However, it is increasingly important to include a broad spectrum of customer and expert insights into our data analysis and engineering tasks: many of these people being non-technical persons with limited exposure to interpreting code and basic data visulizations. As a result, inclusion of non-technical personnel for a project requires a different type of collaborative analysis environment.

I recently built a demonstration in Streamlit for signals data and it dawned on me that these type of dashboard environments are great for providing a collaborative exploratory data analysis (EDA) environment. For technical users, it makes you take scripts you would usually put into a notebook and place them into a more structured `.py` file. This is preferred as this will often be what is needed by data management teams to implement specific ETL tasks. For non-technical users, this environment provides simple, interactive tools with easy buttonology to explore dimensions and factors in the data. This allows expert insights to enhance the ETL process, the resultant metadata, and documentation.

So let's take a look on how to quickly and painlessly build a usable dashboard for inter-team collaboration.

## Building the Dashboard

First, create a virtual environment and then install these libraries:

```bash
pip install plotly pandas streamlit
```

You may also want to install jupyter or jupyterlab if you plan to do some preliminary EDA before creating you dataframe function.

Now, let's create a file named `wifi_dash.py` and open in an editor of your choice. First, we'll import the libraries we just installed:

```python
import pandas as pd
import plotly.express as px
import streamlit as st
```

We can then declare our page configuration for the dashboard:

```python
st.set_page_config(page_title="WiFi Dashboard",
                   age_icon=":bar_chart:",
                   layout="wide")
```

Here we're declaring the page title, the favicon, and the page layout. Now let's get some data in here:

```python
@st.cache
def get_data(data):
    wifi = pd.read_json(data)
    wifi_signals = wifi['results'].apply(pd.Series)
    wifi_signals.rename(columns={'trilat':'lat', 'trilong':'lon'}, inplace=True)
    wifi_signals['year'] = pd.to_datetime(wifi_signals['lastupdt'], format="%Y-%m-%d").dt.year
    wifi_signals['count'] = 1
    wifi_signals['dow'] = pd.to_datetime(wifi_signals['lastupdt']).dt.dayofweek
    return wifi_signals

df = get_data("data/wigle_tpa_wifi.json")
```

I chose to use WiGLE WiFi data for my dashboard, but you can adapt this workflow to the data you need to explore.

Okay, let's break this insanity down:

* Before the function, we'll use the Streamlit decorator `@st.cache` that tells Streamlit to locally cache the results of the function.
* Then we'll declare our dataframe function with a single argument for the input data source (add more if your data ingest is more complex).
* We declare our initial data object using Pandas to read a JSON file.
* The next 5 steps perform data transformation we'll need for dashboard functionality.
* We then return the processed dataframe out of the function.
* Finally, we assign the function output to the variable `df` using our raw json file from WiGLE.

Okay, we are now ready to start creating the dashboard. First, let's create our sidebar.

```python
st.sidebar.header("Filter Here:")
city = st.sidebar.multiselect(
        "Select the City:",
        options=df["city"].unique(),
        default=df["city"].unique()
        )

road = st.sidebar.multiselect(
        "Select the Road:",
        options=df["road"].unique(),
        default=df["road"].unique()
        )

postalcode = st.sidebar.multiselect(
        "Select the Zip Code:",
        options=df["postalcode"].unique(),
        default=df["postalcode"].unique()
        )

housenumber = st.sidebar.multiselect(
        "Select the House Number:",
        options=df["housenumber"].unique(),
        default=df["housenumber"].unique()
        )

year = st.sidebar.multiselect(
        "Select the Year:",
        options=df["year"].unique(),
        default=df["year"].unique()
        )

encryption = st.sidebar.multiselect(
        "Select the Encyption:",
        options=df["encryption"].unique(),
        default=df["encryption"].unique()
        )

ssid = st.sidebar.multiselect(
        "Select the SSID:",
        options=df["ssid"].unique(),
        default=df["ssid"].unique()
        )

df_selection = df.query(
        "city == @city & road == @road & postalcode == @postalcode & housenumber == @housenumber & year == @year & encryption == @encryption & ssid == @ssid"
        )             
```

So here's what happened:

* We used `st.sidebar.header` to declare our sidebar title.
* Then we use `st.sidebar.multiselect` to declare several variables as selection options that will each be populated with their unique values found in the data field.
* After each variable (e.g., city, year, encryption, etc.), we declare the `df_selection` variable which queries our dataframe object using the Pandas query method.

You can now go ahead and do a quick test run of the streamlit app! Below the df\_selection object, type:

```python
st.dataframe(df_selection)
```

Then, inside your active virtual environment, execute:

```bash
streamlit run wifi_dash.py
```

Alright! Now we're cooking! You should see the dataframe and a sidebar that when changed leaves only the rows based on the remaining selections.

Let's create our main dashboard starting with the main title:

```python
st.title("Dashboard - WiGLE WiFi Signals")
st.markdown("##")
st.markdown("""---""")
```

Now let's create a section for three descriptive data variables:

```python
unique_ssid = len(df_selection['ssid'].value_counts())
unique_house = len(df_selection['housenumber'].value_counts())
unique_roads = len(df_selection['road'].value_counts())
```

After declaring these three, lets put each into its own column using `st.columns`:

```python
left_column, middle_column, right_column = st.columns(3)
with left_column:
    st.subheader("Unique SSIDs:")
    st.subheader(f"{unique_ssid:,}")
with middle_column:
    st.subheader("Unique House Numbers Found")
    st.subheader(f"{unique_house:,}")
with right_column:
    st.subheader("Unique Roads")
    st.subheader(f"{unique_roads:,}")

st.markdown("""---""")
```

Using `with` statements, we pushed each variable into its column as defined by its column name. We also get to declare a title and it's interactive variable using `st.subheader` and a little f-string awesomeness. Now if we view the dashboard, you should see the title and a three-column stats section.

Now let's create a charts section:

```python
st.title("Charts")
st.markdown("##")
```

After the section title, let create a chart:

```python
signals_per_year = df_selection.groupby(by=['year']).sum()[['count']]
fig_year_signals = px.bar(
    signals_per_year,
    x=signals_per_year.index,
    y='count',
    title='Signals per Year',
    color_discrete_sequence=["#0083B8"] * len(signals_per_year),
    template="plotly_white",
    )
fig_year_signals.update_layout(
    xaxis=dict(tickmode="linear"),
    plot_bgcolor="rgba(0,0,0,0)",
    yaxis=(dict(showgrid=False)),
)                 
```

So here is what we did:

* First, we group the variables established by the group. In this case, we wanted to see signals per year within our data selection.
* We then use the Pandas groupby method to build a signals per year dataframe.
* This grouped dataframe is then used to create a Plotly bar chart.
* We then update the chart object with some custom styling.

Let's create a second chart using the same method:

```python
signals_per_dow = df_selection.groupby(by=['dow']).sum()[['count']]
fig_dow_signals = px.bar(
    signals_per_dow,
    x=signals_per_dow.index,
    y='count',
    title='Signals per Day of Week (Mon = 0, Sun = 6)',
    color_discrete_sequence=["#0083B8"] * len(signals_per_year),
    template="plotly_white",
    )
fig_dow_signals.update_layout(
    xaxis=dict(tickmode="linear"),
    plot_bgcolor="rgba(0,0,0,0)",
    yaxis=(dict(showgrid=False)),
)
```

We can now take our two charts and put them side-by-side, similar to how we created the stats section:

```python
left_column, right_column = st.columns(2)
left_column.plotly_chart(fig_year_signals, use_container_width=True)
right_column.plotly_chart(fig_dow_signals, use_container_width=True)

st.markdown("""---""")
```

Notice how we used the `plotly_chart` method within `st.columns` which helps with size and style of the charts as we place them in the dashboard.

Whew! Now it's looking like a real dashboard... but we can do more!

WiGLE comes with geospatial (X,Y) coordinates, so let's create a map!

```python
st.title("Map of Filtered Signals")
st.markdown("##")
st.map(df_selection)

st.markdown("""---""")
```

It's really that simple to create a dot map in Streamlit.

Finally, let's move our raw dataframe view from above to the bottom of the dashboard: we want our visualizations to take center stage, but still want people to see the raw data if needed.

```python
st.title("Raw Data")
st.markdown("##")
st.dataframe(df_selection)

st.markdown("""---""")
```

That's pretty much it! If you want to add some additional styling, you can create a `.streamlit` folder and then inside the folder a `config.toml` file with custom styling:

```python
[theme]
# Primary accent color for interactive elements.
primaryColor = "#E694FF"

# Background color for the main content area.
backgroundColor = "#00172B"

# Background color used for the sidebar and most interactive widgets.
secondaryBackgroundColor = "#0083B8"

# Color used for almost all text.
textColor = "#FFF"

# Font family for all text in the app, except code blocks. One of "sans serif", "serif", or "monospace".
# Default: "sans serif"
font = "sans serif"
```

We can then go back to our `wifi_dash.py` file and add this to the end:

```python
hide_st_style = """
                <‍style>
                #MainMenu {visibility: hidden;}
                footer {visibility: hidden;}
                header {visibility: hidden;}
                <‍style/>
                """

st.markdown(hide_st_style, unsafe_allow_html=True)
```

So this is now running locally, but what if you want to share it?!

Streamlit offers a "free" deployment solution through their website to host up to three dashboards.

* Go to [Streamlit](https://share.streamlit.io/ "https://share.streamlit.io/") to login with your Google or GitHub account.

## What We Did

As you can see, this is much easier to give to a non-technical person than a Jupyter notebook or a Python script: individuals or groups can view this dashboard, ask questions, and refine their data models collaboratively. Once insights and tasks are identified, changes can easily be made and the dashboard updated quickly.

In terms of data management (my current application space for this), I can generate interactive dashboards as part of the initial ETL process. This tooling provides data owners and data stewards a way to convey nuanced insights and requirements generated through EDA to data teams: a blog for another time perhaps 😄.

## References

* [Streamlit Docs](https://docs.streamlit.io/ "https://docs.streamlit.io/")
* [WiGLE](https://www.wigle.net/ "https://www.wigle.net/")
