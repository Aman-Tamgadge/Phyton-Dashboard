# Phyton-Dashboard
import pandas as pd
import dash
from dash import dcc, html
import plotly.express as px

# Load your dataset
df = pd.read_csv("chatgpt_reviews.csv")

# Example: assume your dataset has columns like 'rating', 'date', 'platform'
# Adjust column names based on your actual file
df['date'] = pd.to_datetime(df['date'])

# Initialize the app
app = dash.Dash(__name__)

# Layout
app.layout = html.Div([
    html.H1("ChatGPT Reviews Dashboard"),

    # Dropdown for platform filter
    dcc.Dropdown(
        id='platform-filter',
        options=[{'label': p, 'value': p} for p in df['platform'].unique()],
        value=df['platform'].unique()[0],
        clearable=False
    ),

    # Line chart of ratings over time
    dcc.Graph(id='rating-trend'),

    # Bar chart of average ratings by platform
    dcc.Graph(
        figure=px.bar(
            df.groupby('platform')['rating'].mean().reset_index(),
            x='platform', y='rating',
            title="Average Rating by Platform"
        )
    )
])

# Callbacks for interactivity
@app.callback(
    dash.dependencies.Output('rating-trend', 'figure'),
    [dash.dependencies.Input('platform-filter', 'value')]
)
def update_trend(selected_platform):
    filtered = df[df['platform'] == selected_platform]
    fig = px.line(filtered, x='date', y='rating',
                  title=f"Ratings Over Time ({selected_platform})")
    return fig

if __name__ == '__main__':
    app.run_server(debug=True)
