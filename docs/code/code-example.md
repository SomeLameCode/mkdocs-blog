code example


```  py title="generate normal distribution" linenums="1"
from numpy.random import Generator, PCG64
import pandas as pd
from scipy.stats import iqr

def generate_data(
    distribution: str = "normal", generator=None, drift_magnitude: float = 0, size: int = 100000
) -> pd.Series:
    """Generates a pandas series with samples drawn from a distribution (normal, pareto or uniform). The internal parameters
    for each distributions are fixed. You can specify the number of samples you want and also if a drift of a certain magnitude is
    to be injected. The drift magnitude is a ratio of the distribution's interquartile range.

    """
    if generator is None:
        generator = Generator(PCG64(12345))

    if distribution == "normal":
        sample = generator.standard_normal(size)
    elif distribution == "pareto":
        a,m = 7.,2.
        sample = (generator.pareto(a, size) + 1) * m
    elif distribution == "uniform":
        sample = generator.uniform(-5,5,size)
    else:
        raise ValueError("Distribution not found.")
    offset = (iqr(sample)) * drift_magnitude
    drifted_sample = sample + offset
    return pd.Series(drifted_sample)
```