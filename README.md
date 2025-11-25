# Sci-forge
We can do physical world experiment in digital world. It focused for  students
### Fully Modular Code (Copy-Paste Ready)

#### 1. `src/entities/experiment.ts`
```ts
// Pure data — no React, no side effects
export interface Apparatus {
  id: string
  type: 'beaker' | 'spring' | 'lens' | 'dna' | 'wire'
  x: number
  y: number
  props?: Record<string, any>
}

export interface Experiment {
  id: string
  title: string
  subject: 'Physics' | 'Chemistry' | 'Biology'
  apparatus: Apparatus[]
  createdBy?: string
  isPublic: boolean
  thumbnail?: string
}
import { GoogleGenerativeAI } from '@google/generative-ai'
import { Experiment } from '@/entities/experiment'

const genAI = new GoogleGenerativeAI(process.env.EXPO_PUBLIC_GEMINI_KEY!)

const model = genAI.getGenerativeModel({
  model: 'gemini-1.5-flash',
  generationConfig: { responseMimeType: 'text/plain' },
})

export class GeminiService {
  static async explain(experiment: Experiment, question?: string): Promise<string> {
    const prompt = question
      ? `As a world-class science teacher, answer this student question about the experiment:\nQuestion: "\( {question}"\nExperiment: \){experiment.title}\nBe clear, exciting, and accurate.`
      : `Explain this \( {experiment.subject} experiment like a Google NotebookLM podcast — warm, storytelling style, perfect for students: \){experiment.title}`

    const result = await model.generateContent(prompt)
    return result.response.text()
  }

  static async generateAudioScript(experiment: Experiment): Promise<string> {
    const result = await model.generateContent(
      `Write a 2-minute engaging podcast script (like NotebookLM) for this experiment: ${experiment.title}. Use simple language, excitement, and sound effects cues.`
    )
    return result.response.text()
  }
}
import { useState, useEffect } from 'react'
import { Experiment } from '@/entities/experiment'
import { experimentService } from '../services/experimentService'

export function useExperiment(id: string | string[]) {
  const [experiment, setExperiment] = useState<Experiment | null>(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    const unsub = experimentService.subscribe(id as string, setExperiment, setLoading)
    return () => unsub()
  }, [id])

  return { experiment, loading, update: experimentService.update }
}
import { db } from '@/services/firebase'
import { collection, doc, onSnapshot, updateDoc } from 'firebase/firestore'
import { Experiment } from '@/entities/experiment'

class ExperimentService {
  private collection = collection(db, 'experiments')

  subscribe(id: string, onData: (exp: Experiment) => void, onLoading: (l: boolean) => void) {
    onLoading(true)
    return onSnapshot(doc(this.collection, id), (snap) => {
      onLoading(false)
      if (snap.exists()) onData(snap.data() as Experiment)
    })
  }

  async update(id: string, updates: Partial<Experiment>) {
    await updateDoc(doc(this.collection, id), updates)
  }
}

export const experimentService = new ExperimentService()
import { View } from 'react-native'
import { useLocalSearchParams } from 'expo-router'
import { LabCanvas } from '../components/LabCanvas'
import { RealtimeGraph } from '../components/RealtimeGraph'
import { AIExplanationPanel } from '@/features/ai/components/AIExplanationPanel'
import { GlassHeader } from '@/shared/ui/GlassHeader'
import Animated, { FadeIn, Layout } from 'react-native-reanimated'
import { useExperiment } from '../hooks/useExperiment'

export default function LabScreen() {
  const { id } = useLocalSearchParams()
  const { experiment, loading } = useExperiment(id)

  if (loading || !experiment) return <LoadingState />

  return (
    <View className="flex-1 bg-black">
      <GlassHeader title={experiment.title} subject={experiment.subject} />

      <Animated.View entering={FadeIn} layout={Layout.springify()} className="flex-1">
        <LabCanvas experiment={experiment} />
      </Animated.View>

      <Animated.View entering={FadeIn.delay(400)} className="absolute bottom-0 left-0 right-0">
        <RealtimeGraph experimentId={id as string} />
        <AIExplanationPanel experiment={experiment} />
      </Animated.View>
    </View>
  )
}
import { BlurView } from 'expo-blur'
import Animated from 'react-native-reanimated'

export function GlassCard({ children, intensity = 80 }: { children: React.ReactNode; intensity?: number }) {
  return (
    <BlurView intensity={intensity} tint="dark" className="rounded-3xl overflow-hidden border border-white/10">
      <Animated.View className="bg-black/40 backdrop-blur-xl p-5">
        {children}
      </Animated.View>
    </BlurView>
  )
}
{
  "dependencies": {
    "expo": "~51.0.0",
    "react-native-reanimated": "~3.8.0",
    "react-native-skia": "1.2.0",
    "@shopify/react-native-skia": "1.2.0",
    "firebase": "^10.12.0",
    "@google/generative-ai": "^0.21.0",
    "expo-router": "~3.7.0",
    "expo-blur": "~13.0.0",
    "moti": "^0.29.0",
    "victory-native": "^41.0.0"
  }
}git clone https://github.com/sciforge/modular-v2.git
cd modular-v2
npm install
cp .env.example .env
npx expo start --clear
