---
layout : post
tags : test
category : test
---

this is a code highlight test

{% highlight cpp %}

#include <vector>
#include <boost/function.hpp>
#include "AudioFeatureExtraction.h"
#include "xprojthrift.h"        // createServer
#include "audio_feature_extract.h"
//#include "config_manager.h"
#include "xprojparams.h"
#include <stdexcept>
#include <boost/thread.hpp>

#ifdef PROFILE_PERFORMANCE
#include <chrono>
#endif

using namespace std;
using namespace xproj::thrift;
using namespace  ::xproj::audio_feature_extraction;
using namespace xproj::params;
using namespace xproj::audio;   // audioFeature
using boost::shared_ptr;
using std::vector;

static std::vector<AudioFeature> _extractAudioFeatures(const std::vector<int32_t>& samples,
                                                boost::shared_ptr<audio_feature_extract> extractor,
                                                const xproj::params::FeatureExtractionParams& p) {

  const size_t maxesPerSpan = p.maxesPerSecond * p.spanSize;
  size_t samplesPerSpan = p.spanSize * p.sampleRate;

  // TODO check allocation
  // afeature* spanFeatures = new afeature[maxesPerSpan];
  afeature spanFeatures[maxesPerSpan];

  uint16_t tsamples[samplesPerSpan];
  for (size_t i = 0; i < samples.size(); ++i) {
    tsamples[i] = samples[i];
  }

  int featuresExtracted = extractor->GenerateFeature(spanFeatures, maxesPerSpan, (short*)tsamples, samplesPerSpan, eadv);

  std::vector<AudioFeature> features(featuresExtracted);
  for (int i = 0; i < featuresExtracted; ++i) {
    features[i].pointHash = spanFeatures[i].pointhash;
    features[i].timePoint = spanFeatures[i].timepoint;
  }
  // delete [] spanFeatures;
  return features;
}

typedef std::vector<std::vector<int32_t> >::const_iterator SampleIter;
typedef std::vector<std::vector<AudioFeature> >::iterator ResultInputIter;

static void extractAudioFeaturesThread(SampleIter sampleOrigin,
                                       SampleIter begin,
                                       SampleIter end,
                                       ResultInputIter resultIter,
                                       xproj::params::FeatureExtractionParams p){

  boost::shared_ptr<audio_feature_extract>
    localExtractor(new audio_feature_extract(p.sampleRate,
                                             p.nChannels,
                                             p.spanSize,
                                             p.dens,
                                             p.spreadFactor,
                                             p.maxesPerSecond,
                                             p.startLag));

  if (begin != sampleOrigin){
    _extractAudioFeatures(*(begin - 1), localExtractor, p);
  }

  for (SampleIter sampleIter = begin; sampleIter != end; ++sampleIter) {
    *resultIter++ = _extractAudioFeatures(*sampleIter, localExtractor, p);
  }

}


class AudioFeatureExtractionHandler : virtual public AudioFeatureExtractionIf {
public:

  AudioFeatureExtractionHandler() { }

  void extractAudioFeatures(std::vector<AudioFeature> & _return,
                            const std::vector<int32_t> & samples,
                            const  ::xproj::params::ExtractionType::type t) {

    const FeatureExtractionParams p = getParams(t);

    static boost::shared_ptr<audio_feature_extract>
      extractor(new audio_feature_extract(p.sampleRate,
                                          p.nChannels,
                                          p.spanSize,
                                          p.dens,
                                          p.spreadFactor,
                                          p.maxesPerSecond,
                                          p.startLag));
    // const size_t maxesPerSpan = p.maxesPerSecond * p.spanSize;
    size_t samplesPerSpan = p.spanSize * p.sampleRate;

    if (samplesPerSpan != samples.size()) {
      throw invalid_argument("samplesPerSpan not equal to samples.size()");
    }

    _return = _extractAudioFeatures(samples, extractor, p);
    printf("single mode extracted %d features\n", _return.size());
  }

  void extractAudioFeaturesBatch(std::vector<std::vector<AudioFeature> > & _return,
                                 const std::vector<std::vector<int32_t> > & vecSamples,
                                 const  ::xproj::params::ExtractionType::type t) {

    printf("Accepted request, vecSamples.size(): %d\n", vecSamples.size());

#ifdef PROFILE_PERFORMANCE
    auto t_start = std::chrono::high_resolution_clock::now();
#endif

    const FeatureExtractionParams p = getParams(t);
    boost::shared_ptr<audio_feature_extract>
      batchExtractor(new audio_feature_extract(p.sampleRate,
                                               p.nChannels,
                                               p.spanSize,
                                               p.dens,
                                               p.spreadFactor,
                                               p.maxesPerSecond,
                                               p.startLag));


    if (vecSamples.size() <= 7) {
      // won't bother to create any extra threads
      _return = std::vector<std::vector<AudioFeature> >(vecSamples.size());
      for (size_t i = 0; i < vecSamples.size(); ++i) {
        _return[i] =  _extractAudioFeatures(vecSamples[i], batchExtractor, p);
      }
    } else {
      // let us spawn 8 threads
      _return = std::vector<std::vector<AudioFeature> >(vecSamples.size());
      boost::thread_group tgroup;
      size_t N_THREADS = 8;
      size_t tasksPerThread = vecSamples.size() / N_THREADS;
      SampleIter taskBegin, taskEnd;
      ResultInputIter resultIter;
      for (size_t i = 0; i < N_THREADS; ++i) {
        taskBegin = vecSamples.begin() + i * tasksPerThread;
        taskEnd = taskBegin + tasksPerThread;
        resultIter = _return.begin() + i * tasksPerThread;
        tgroup.create_thread(boost::bind(&extractAudioFeaturesThread,
                                         vecSamples.begin(),
                                         taskBegin,
                                         taskEnd,
                                         resultIter,
                                         boost::ref(p)));
      }
      tgroup.join_all();

      for (size_t i = N_THREADS * tasksPerThread; i < vecSamples.size(); ++i) {
        _extractAudioFeatures(vecSamples[i - 1], batchExtractor, p);
        _return[i] =  _extractAudioFeatures(vecSamples[i], batchExtractor, p);
      }
    }

#ifdef PROFILE_PERFORMANCE
    auto t_end = std::chrono::high_resolution_clock::now();
    printf("features extracted in %d: ms\n", std::chrono::duration_cast<std::chrono::milliseconds>(t_end - t_start).count());
#endif

    printf("feature extraction done\n");

  }

};

int main(int argc, char **argv) {
  // on constructing, ZookeeperRegister will automatically register
  // the current machine;
  // on destruction, ZookeeperRegister will automatically unregister
  // the current machine

  // CAUTION! you must use a head-allocated ZookeeperRegister here;
  // otherwise ZookeeperRegister will be destroyed after server->serve(),
  // and the thrift server will be unregstered.
 // boost::shared_ptr<xproj::configure::ZookeeperRegister<xproj::configure::FeatureServerConf> > t(new xproj::configure::ZookeeperRegister<xproj::configure::FeatureServerConf>());

  //start to serve
  //boost::shared_ptr<TNonblockingServer> server = createServer<AudioFeatureExtractionHandler, AudioFeatureExtractionProcessor>(xproj::configure::FeatureServerConf::port, 15);
  boost::shared_ptr<TNonblockingServer> server = createServer<AudioFeatureExtractionHandler, AudioFeatureExtractionProcessor>(9000, 15);
  server->serve();

  return 0;
}

{%endhighlight%}
